# Experiment A — INC-002

## Experiment Details

| Parameter | Value |
|---|---|
| Experiment ID | EXP-A |
| Experiment Name | Basic Prompting |
| Incident ID | INC-002 |
| Pipeline | orders_incremental_load |
| Model | Claude Sonnet 5.5 |
| Effort | Medium |
| Prompt Type | Basic Prompting |
| Ground Truth Provided | No |
| Temperature | Default |

## Claude Output

Summary

The load "succeeded" but 1,200 records (10%) were lost, and the loss happened inside the transformation step, not during extraction or loading. This is a silent data-loss incident: the job reported success while the destination is incomplete.

Where the Records Were Lost
Stage	Count	Delta
Source extraction	12,000	n/a
Transformation output	10,800	-1,200
Destination load	10,800	0

Extraction and load counts are internally consistent with their neighbors. The drop occurs only between 04:08 and 04:12. The destination matches the transformation output exactly, so the load step is not losing data. The failure is upstream of it.

Most Likely Root Cause

The transformation step dropped 1,200 records without raising an error, most likely through filtering or rejection logic that discards rows silently. The exact rate (exactly 10.0%) is notable: a round number suggests a systematic cause (a specific condition matching a fixed slice of the data, or a batch/partition boundary) rather than random corruption.

What Might Have Gone Wrong
Data quality rejections. Rows with nulls in required fields, invalid formats (dates, currency, IDs), or type-cast failures were dropped by the transform with no error surfaced. This is the most common cause.
Join issues. An inner join against a dimension or lookup table (customers, products) drops orders with no matching key, for example new customers or products not yet loaded into the dimension. This is a very common cause in order pipelines.
Deduplication logic. The transform may remove duplicates (for example DISTINCT or ROW_NUMBER() = 1) on a key that is not truly unique, collapsing legitimate distinct orders. Incremental loads that overlap the previous watermark can also legitimately contain duplicates.
Overly aggressive filters. A WHERE clause (status, date range, test-order exclusion, region filter) that excludes rows which should be kept, possibly after a recent business-rule change.
Schema or upstream data change. A new value, format, or column change in the source that the transform doesn't handle, causing rows to fail validation and be skipped.
Partial or failed partition/batch (less likely). If transformation is parallelized, one worker or partition may have failed or timed out and been swallowed, which could explain the round 10% figure.
Error handling that masks failures. A try/except or "skip bad records" setting that logs nothing, so the job still reports success.

Important caveat: it's possible some or all of the 1,200 drops are intentional (for example, filtering out cancelled or test orders). The data provided can't distinguish intended filtering from a bug. That is the first thing to verify.

Gap in the data: rejected_record_details was not provided. This is the key missing artifact. If the pipeline writes a reject/dead-letter table or log, it would likely identify the cause directly.

Troubleshooting Steps

1. Identify exactly which records were dropped

Anti-join the extracted source set against the destination on the order key to list the 1,200 missing records.
Look for common traits among them: null columns, order status, customer/product IDs, timestamps, region, source file or partition.

2. Check for a reject or dead-letter output

Look for a rejects table, quarantine bucket, or skipped-row logs from the transformation job (Spark/dbt/SQL logs, DLQ).
Check whether the transform has a "drop malformed" or "permissive" mode enabled.

3. Review the transformation logic

Inspect filters, inner joins, dedup logic, and type casts.
Run the transform on the 12,000-row extract step by step, counting rows after each stage to find the exact operation that removes them.

4. Check for recent changes

Review recent changes to transformation code, business rules, lookup/dimension tables, and the source schema or data contract.
Compare against previous runs: was the 10% drop rate normal historically, or new?

5. Check for join key coverage

Verify that all customer and product keys in the missing orders exist in the dimension tables, and check load ordering (were dimensions refreshed before this run?).

6. Check for duplicates and watermark overlap

Count distinct order keys in the 12,000 extract versus total rows. Confirm the incremental watermark isn't re-extracting previously loaded records.
Remediation
If drops were unintended: fix the transform logic or reference data, then reprocess the missing 1,200 records. Use an idempotent upsert/merge keyed on order ID to avoid creating duplicates. Do not blindly rerun the full batch unless the load is idempotent.
If drops were intentional: document the rule, and update the reconciliation check so expected counts account for it.
Notify downstream consumers (reporting, finance, fulfillment) that orders data for this window may be incomplete until the backfill is done. Orders data often feeds revenue reporting, so the impact may be significant.
Assess the watermark: confirm the next incremental run won't skip the missing records. If the watermark advanced past them, they will never be picked up automatically.
Preventive Recommendations
Never drop silently. Route rejected rows to a quarantine table with a rejection reason code, and log counts per rule.
Fail or alert on thresholds. A 10% variance should fail the job or page someone, not complete as "WITH_WARNING." Set tolerances (for example, >0.5% unexplained loss fails the run).
Stage-level reconciliation. Emit row counts at every stage (in, out, rejected) and assert in = out + rejected.
Data contracts and schema checks on the source to catch new values or format changes early.
Row-level reconciliation (key-based, not just counts) in post-load validation so you can identify the exact missing records automatically.
Confidence Note

Confidence is high that the loss occurred in the transformation step, since the counts rule out extraction and load. Confidence is low on the specific mechanism (bad data vs. join vs. filter vs. dedup) until the rejected-record details or a stage-by-stage row count are available. It is also not yet established whether the loss is a bug or intended behavior.

## Response Metadata

- Incident: INC-002
- Experiment: A — Basic Prompting
- Model: Claude Sonnet 5.5
- Effort: Medium
- Original response preserved without modifications.
- Ground truth was not shared with Claude.