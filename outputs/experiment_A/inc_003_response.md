# Experiment A — INC-003

## Experiment Details

| Parameter | Value |
|---|---|
| Experiment ID | EXP-A |
| Experiment Name | Basic Prompting |
| Incident ID | INC-003 |
| Pipeline | customer_profile_ingestion |
| Model | Claude Sonnet 5.5 |
| Effort | Medium |
| Prompt Type | Basic Prompting |
| Ground Truth Provided | No |
| Temperature | Default |

## Claude Output

Summary

The job failed at schema validation, before any data was read, so 0 of 8,000 records were processed. The source file arrived with a new column, loyalty_tier, and the pipeline's strict validation rejected the file. This is a schema drift incident: the data itself is probably fine, but the pipeline's contract with the source is out of date.

Schema Comparison
Column	Expected	Received	Status
customer_id	yes	yes	OK
name	yes	yes	OK
email	yes	yes	OK
updated_at	yes	yes	OK
loyalty_tier	no	yes	Unexpected

The only difference is one additive column. Nothing is missing, renamed, or reordered, so this is the least dangerous kind of schema change. The failure came from the validator's strictness, not from corrupt data.

Most Likely Root Cause

The upstream source added loyalty_tier to the customer profile export without the ingestion pipeline being updated or notified. The validation step uses an exact-match (strict) schema check, so any extra column causes a hard failure.

Two things combined to cause this:

A change on the producer side that was not communicated, or was communicated but not coordinated with a pipeline release.
A strict, non-evolving schema policy on the consumer side that treats additive changes the same as breaking ones.
What Might Have Gone Wrong
Uncoordinated upstream release. The source team shipped a feature (a loyalty program, perhaps) and updated the export with no data contract, change notice, or versioning.
No schema evolution policy. The policy is "Not provided," which suggests there is none, or it is undocumented. Without one, the default is to fail on anything unexpected.
Validator too strict for additive changes. Rejecting extra columns is reasonable for catching mistakes, but unhelpful when the new column is benign.
No staging or compatibility test. Nothing detected the upcoming change before it hit production.
Possible downstream implications (to verify). loyalty_tier may contain PII-adjacent or business-sensitive data, or may need to be added to the target table, models, and reports before it is safe to load.

Caveat: I can't tell from the logs whether the column was added intentionally by the source team or by accident (for example, a wrong export template). That is worth confirming first.

Troubleshooting Steps

1. Confirm the change with the source owner

Ask whether loyalty_tier is intentional, permanent, and what it contains (data type, allowed values, nullability, when it is populated).
Confirm whether more schema changes are planned.

2. Inspect the file itself

Check that the 8,000 rows are intact: row count, header parsing, delimiters, quoting, and encoding.
Profile loyalty_tier: null rate, distinct values, and data type. Check that no commas or quotes in that field break the CSV structure.
Compare against yesterday's file to confirm that this is the only difference.

3. Check the target and downstream dependencies

Does the destination table have a loyalty_tier column? If not, a decision is needed: add it, or ignore it.
Check downstream models, views, and exports that use SELECT * or positional column mapping and could break when the column appears.
Check whether data governance or privacy review is needed for the new field.

4. Choose a fix path

Option	When to use	Effect
A. Accept the column	Field is wanted	Add loyalty_tier to the expected schema and target table (nullable), deploy, rerun
B. Ignore the column	Field is not needed yet	Configure the job to select only known columns, rerun
C. Reject the file	Change was a mistake	Ask the source to re-export without it

Option A is the best long-term outcome if the field is legitimate. Option B is the fastest and lowest-risk way to restore service today.

5. Recover

Rerun ingestion for customer_profile_20261125.csv once the fix is in place. Since 0 records were processed, there should be no partial loads, but confirm the target has no partial writes from this run.
Make sure the rerun is idempotent (upsert on customer_id, using updated_at to resolve conflicts).
Check that subsequent runs don't skip files, and that the file is picked up again after the fix and not marked as already processed.
Notify consumers of the delay.
Preventive Recommendations
Define a schema evolution policy. For example: additive nullable columns are auto-accepted with an alert; removed, renamed, or retyped columns fail the pipeline.
Use a data contract with the producer. Require advance notice and versioning for changes, and run contract tests in CI on the producer side.
Make validation tiered. Distinguish severity levels: missing or retyped columns are errors, new columns are warnings. Quarantine unknown columns instead of failing the whole file.
Alert on drift, not just failure. Emit a schema-drift event, so the team knows about changes even when they are handled automatically.
Prefer explicit column lists over SELECT * or positional mapping in transforms and downstream models, so added columns can't break them.
Add a staging/canary check that validates incoming files against the schema before the scheduled production window.
Retry or hold-and-resume behavior. Park the failed file in a pending state, and reprocess it automatically after a schema update, rather than needing a manual rerun.
Severity Note

MEDIUM severity seems appropriate for the current impact (one delayed daily file, no data loss). Consider raising it if downstream consumers have morning deadlines on this data, or if the file is not reprocessed before the next scheduled run, since the profile data would become stale.

Confidence Note

Confidence is high in the root cause: the logs show exactly the mismatch that triggered the error, and the diagnostics confirm it. Confidence is low on the right fix (accept, ignore, or reject) until the source owner confirms the intent and the downstream impact of loyalty_tier is known. The missing schema evolution policy is a process gap, and it explains why a benign change became a pipeline failure.

## Response Metadata

- Incident: INC-003
- Experiment: A — Basic Prompting
- Model: Claude Sonnet 5.5
- Effort: Medium
- Original response preserved without modifications.
- Ground truth was not shared with Claude.