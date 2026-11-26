# Experiment A — INC-004

## Experiment Details

| Parameter | Value |
|---|---|
| Experiment ID | EXP-A |
| Experiment Name | Basic Prompting |
| Incident ID | INC-004 |
| Pipeline | daily_payments_aggregation |
| Model | Claude Sonnet 5.5 |
| Effort | Medium |
| Prompt Type | Basic Prompting |
| Ground Truth Provided | No |
| Temperature | Default |

## Claude Output

Summary

The job processed all 25,000 records, but the destination contains 420 repeated payment_id values. Counts match at every stage, so nothing was lost. The problem is that some payments appear more than once, which can double-count revenue in a payments aggregate. The logs don't show whether the duplicates came from the source or were created by the pipeline, and that is the central question.

What the Counts Tell Us
Stage	Count	Observation
Source extraction	25,000	Matches expected
Transformation	25,000	No rows added or removed
Destination write	25,000	Matches transformation output
Duplicate check	420 repeated payment_id values	Duplicates exist in the destination

Row counts are identical at every stage, which rules out a transform that fans out rows (such as a one-to-many join), since that would inflate the count above 25,000. It also rules out a destination write that appended an extra copy of the batch, since that would roughly double the count to ~50,000. Two cases remain:

The duplicates were already in the source extract, and the pipeline passed them through untouched.
The destination already held rows from an earlier run and the write didn't replace them. This is only consistent with the counts if the 25,000 figure refers to rows written by this run rather than the table's total size, which the logs don't clarify.

Note also that "420 repeated values" is ambiguous. It could mean 420 distinct IDs that appear more than once, or 420 surplus rows. The affected row count is somewhere between 420 and 840 or more, depending on how many copies each ID has.

Most Likely Root Cause

The pipeline has no uniqueness enforcement on payment_id, so duplicates from the source or from an overlapping prior load went straight into the destination. The most probable mechanisms are:

Duplicates in the source extract. Upstream retries, at-least-once delivery from a queue or CDC stream, or payment state changes (authorized → captured → settled) may emit multiple rows per payment_id.
Overlapping extraction windows. If the extraction window or watermark overlaps the previous run, payments already loaded yesterday are pulled again.
Non-idempotent write mode. write_mode is not provided. If the job uses plain INSERT/append rather than MERGE/upsert keyed on payment_id, any re-extracted or reprocessed record becomes a duplicate.
A rerun or retry without cleanup. A previous partial run, a retry, or a backfill may have already written some of the same payments.

The missing write_mode and source_duplicate_count diagnostics are what prevent a firmer conclusion. One query on the source extract would separate the source-duplicate case from the pipeline-created case.

What Might Have Gone Wrong
Append-only writes with no deduplication step and no unique constraint on the destination.
Watermark handling. The watermark might use >= instead of >, be based on a timestamp that is updated retroactively, or not advance after a failed run.
Late-arriving or updated payments re-extracted because their modified timestamp changed.
Data quality check placed after the write. The duplicates were detected at 06:40, after the data was already in the destination, so bad data was exposed for at least 5 minutes before anyone was alerted.
A warning-level response to a financial integrity issue. The job finished as COMPLETED_WITH_WARNING, so downstream consumers may have already used the data.

Caveat: some repeated payment_id values may be legitimate if the table is designed to hold multiple events per payment (for example, a status history). If payment_id is not meant to be unique in this table, the check itself may be misconfigured. Confirm the table's grain first.

Troubleshooting Steps

1. Quantify and characterize the duplicates in the destination

Count distinct duplicated IDs vs. surplus rows: SELECT payment_id, COUNT(*) FROM dest GROUP BY payment_id HAVING COUNT(*) > 1.
Compare the duplicate rows: are they exact copies (identical in every column) or do they differ in status, amount, or timestamp? Exact copies suggest re-ingestion. Differing rows suggest multiple events or state changes.
Look at load metadata (batch ID, load timestamp, source file) to see if the duplicates came from this run or an earlier one.

2. Determine whether the duplicates came from the source

Run the same duplicate count on the raw extract (the missing source_duplicate_count). If about 420 repeats are present there, the source is the origin. If the extract is clean, the pipeline created them.

3. Check the write mode and destination constraints

Review the job config: append vs. overwrite vs. merge/upsert, and any deduplication step.
Check whether the destination has a primary key or unique constraint on payment_id (many warehouses don't enforce these).

4. Review the extraction window and watermark

Compare this run's window against the previous run's. Look for overlap or a stalled watermark.
Check the orchestrator history for retries, reruns, or manual triggers on 11-25 or 11-24.

5. Check upstream behavior

Ask the source team whether retries, replays, or backfills occurred, and whether payment_id is guaranteed unique in the feed.

6. Assess downstream impact

Identify who has already consumed the aggregate (finance dashboards, reconciliation, settlement reports). Because the pipeline aggregates payments, duplicates may have inflated totals, so quantify the amount involved (SUM(amount) over the surplus rows).
Remediation
Contain first: if the table feeds financial reporting, quarantine or flag the affected data, or pause downstream refreshes until the duplicates are resolved.
Fix the data: deduplicate the affected rows using a deterministic rule (for example, keep the latest by updated_at, or the row from the most recent load), after confirming which copy is correct. Back up the table or use a transactional/time-travel approach before deleting anything.
Rebuild the aggregates derived from the affected data, and re-verify totals against the source system of record.
Fix the cause: switch to an idempotent MERGE keyed on payment_id, add a dedup step in the transform, or correct the watermark logic, depending on what steps 2-4 reveal.
Notify finance and any other downstream consumers if reports or aggregates were published with the duplicates.
Preventive Recommendations
Idempotent writes: use upsert/merge on a business key, so reruns and overlaps can't create duplicates.
Pre-publish quality gates: run the uniqueness check on a staging table and block promotion to production on failure (write-audit-publish pattern), rather than checking after the write.
Escalate by severity: treat duplicate keys in a payments table as a failure, not a warning, or at least gate downstream consumption on the check passing.
Enforce or assert uniqueness with constraints where supported, or with dbt/Great Expectations-style tests on payment_id.
Source-side checks: record duplicate counts at extraction so the origin can be identified immediately.
Window hygiene: use non-overlapping windows or a watermark with a high-water mark that is stored transactionally.
Reconciliation: compare destination totals (count and sum of amounts) against the source of record after each run.
Confidence Note

Confidence is high that the duplicates are not a fan-out or lost-data problem, since the counts match at every stage, and that the pipeline lacks effective uniqueness protection. Confidence is low on whether the duplicates originated in the source, an overlapping window, or a non-idempotent write, because write_mode and source_duplicate_count were not provided. Running the duplicate query on the raw extract and reviewing the write configuration should settle this quickly.

## Response Metadata

- Incident: INC-004
- Experiment: A — Basic Prompting
- Model: Claude Sonnet 5.5
- Effort: Medium
- Original response preserved without modifications.
- Ground truth was not shared with Claude.