# Experiment A — INC-005

## Experiment Details

| Parameter | Value |
|---|---|
| Experiment ID | EXP-A |
| Experiment Name | Basic Prompting |
| Incident ID | INC-005 |
| Pipeline | sales_warehouse_load |
| Model | Claude Sonnet 5.5 |
| Effort | Medium |
| Prompt Type | Basic Prompting |
| Ground Truth Provided | No |
| Temperature | Default |

## Claude Output

Summary

The extraction task reported success with 30,000 records, but the transformation task, three minutes later, could not find its input at s3://analytics-staging/sales/2026/11/25/input.parquet. Both retries failed the same way, so 0 records were loaded. This is a handoff failure between extraction and transformation: the data was extracted, but the transformation task cannot find it where it expects it. The logs do not say whether the object was never written, was written somewhere else, or was written and later removed.

Timeline
Time	Event	What it tells us
07:00	Extraction completed: 30,000 records	Upstream reports success, but not where the output went
07:03	Transformation started	3-minute gap after extraction
07:04	Lookup of .../2026/11/25/input.parquet returns NOT_FOUND	The object is absent at the expected path
07:05, 07:08	Retries 1 and 2 fail identically	The object is not simply delayed, or it is at least not appearing within 4 minutes
07:10	Pipeline marked FAILED	

The deciding fact is that upstream_output_location was not provided. The path the transformation expects and the path the extraction wrote to cannot be compared from this data.

Most Likely Root Cause

The extraction task's output never landed at the path the transformation task reads from. "Extraction completed" confirms the task finished, not that its output was written to that exact key. The most probable explanations are:

Path or naming mismatch. The extraction wrote to a different bucket, prefix, date partition, or filename. Examples include part-00000-*.parquet files under a directory instead of a single input.parquet, a different environment bucket, or a recent change to the naming convention on one side only.
Write failed or was incomplete but was reported as success. The extract may have failed on the S3 write (permissions, multipart upload not completed, task exiting before the flush) while the task still returned success.
Object deleted or overwritten after it was written. A lifecycle rule, a cleanup job, or a concurrent or earlier run could have removed it between 07:00 and 07:04.
What Might Have Gone Wrong
Configuration drift. The output path and input path are defined separately in the extraction and transformation configs, and one was changed.
Date partition mismatch. The two tasks may compute 2026/11/25 differently (UTC vs. local time, run date vs. data date). This is less likely at 07:00 on this date, but worth ruling out.
Misleading success signal. The orchestrator marks extraction complete based on task exit status, with no check that the output object exists.
Environment mix-up. The extraction may have written to a prod or dev bucket while transformation reads from analytics-staging.
Storage-layer interference. Lifecycle expiration rules, bucket policy changes, or a cleanup job targeting the staging prefix.
Permissions masking as not-found (less likely). S3 normally returns "access denied" for permission problems, but a client library or wrapper may translate errors into a generic FileNotFoundError, and S3 returns 404 instead of 403 when the caller lacks s3:ListBucket. Check that the error is a true 404.
Read-after-write timing (unlikely). S3 is strongly consistent for new objects, so this would only apply to an intermediate layer or non-S3 storage behind the path.

Caveat: the 30,000 figure comes from the extraction task's own report. It has not been verified against any stored object, so the data may not exist anywhere yet.

Troubleshooting Steps

1. Check what actually exists in S3

List the prefix: aws s3 ls s3://analytics-staging/sales/2026/11/25/. Also list parent prefixes (sales/2026/11/, sales/).
Look for near matches: different filenames, part files, other date folders, or a misplaced input.parquet.
If versioning is on, check for delete markers or previous versions of the key.
Check for incomplete multipart uploads on the bucket.

2. Find where extraction wrote its output

Read the extraction task's logs and config for the output location and any write or upload messages (PutObject, bytes written, final key).
Compare the output path template with the transformation's input path template.

3. Trace the object's history

Check CloudTrail S3 data events (or S3 server access logs) for PutObject, CompleteMultipartUpload, and DeleteObject on the key between 07:00 and 07:04. If there was no PutObject, the write never happened. If there was a DeleteObject, find the caller.
Review lifecycle rules and cleanup jobs on the bucket or prefix.

4. Check for recent changes

Review recent deployments or config changes to either task, and changes to the bucket, IAM roles, or bucket policy.
Compare against yesterday's successful run: the path, filename, and write pattern.

5. Verify identity and access

Confirm the transformation role has s3:GetObject and s3:ListBucket on the prefix, and that the extraction role has s3:PutObject.
Reproduce the read manually with the transformation task's credentials to see whether the error is 404 or 403.

6. Check the environment

Confirm both tasks target the same bucket and environment, and check orchestrator variables and parameters for the run.
Remediation
If the object exists elsewhere: correct the path configuration, then rerun the transformation task only. Do not rerun extraction unnecessarily.
If the object was never written or was deleted: rerun extraction, then transformation, and verify the object exists before the transformation starts. Check that the load is idempotent, so a rerun cannot create duplicates if the earlier run partially loaded anything. Here 0 records were processed, so risk is low.
Check the watermark or incremental state first. If extraction advanced its watermark or bookmark on "success," a naive next run could skip these 30,000 records. Confirm the state before rerunning, and reset it if needed.
Notify downstream consumers. The CRITICAL severity suggests sales reporting depends on this load, so communicate the delay and an ETA.
Fix the root cause before the next scheduled run, so the same failure does not repeat.
Preventive Recommendations
Verify output on write. After extraction, check that the output object exists and matches the expected size or row count before marking the task successful.
Pass the path explicitly. Have the extraction task emit its output location (for example via orchestrator XCom or task output), and have transformation consume that value instead of rebuilding the path independently.
Single source of truth for paths. Define bucket, prefix, and naming in one shared config.
Don't blindly retry non-retryable errors. A missing object is unlikely to appear on retry within minutes. Fail fast, with a short wait only if there is a known propagation or upstream-lag reason, and alert with the exact path and a prefix listing in the message.
Add a pre-flight check at the start of transformation that verifies inputs exist and lists nearby keys on failure to speed up diagnosis.
Review lifecycle and cleanup rules so staging data is not removed while a run is in flight.
Alert on the gap between extraction completion and the downstream task, and include extraction's reported output location in alerts.
Add a contract or smoke test that checks writer and reader path conventions in CI.
Confidence Note

Confidence is high that the failure is an input-handoff problem and not a data-volume or transformation-logic issue, since the transformation never got to read any data. Confidence is low on the specific mechanism (path mismatch vs. failed write vs. deletion) because upstream_output_location was not provided. A prefix listing in S3 plus the extraction task's write logs should narrow it down within minutes, and CloudTrail data events will settle it if those are enabled.

## Response Metadata

- Incident: INC-005
- Experiment: A — Basic Prompting
- Model: Claude Sonnet 5.5
- Effort: Medium
- Original response preserved without modifications.
- Ground truth was not shared with Claude.