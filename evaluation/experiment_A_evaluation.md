# Experiment A — Basic Prompting Evaluation

## 1. Evaluation Overview

| Parameter | Value |
|---|---|
| Experiment ID | EXP-A |
| Experiment Name | Basic Prompting |
| Model | Claude Sonnet 5.5 |
| Effort | Medium |
| Incidents Evaluated | 5 |
| Maximum Score | 125 |
| Evaluation Rubric | evaluation_rubric.md |

All five Claude responses were evaluated against their corresponding independent ground-truth references.

## 2. Evaluation Results

| Incident | Accuracy | Evidence | Hallucination Avoidance | Usefulness | Uncertainty | Total |
|---|---:|---:|---:|---:|---:|---:|
| INC-001 | 5 | 5 | 4 | 5 | 5 | 24 |
| INC-002 | 5 | 5 | 4 | 5 | 5 | 24 |
| INC-003 | 5 | 5 | 4 | 5 | 5 | 24 |
| INC-004 | 5 | 4 | 3 | 5 | 5 | 22 |
| INC-005 | 5 | 5 | 4 | 5 | 5 | 24 |
| **Total** | **25** | **24** | **19** | **25** | **25** | **118** |

## 3. Incident-Level Findings

### 3.1 INC-001 — Database Connection Timeout

**Score: 24/25**

Claude correctly identified network connectivity as the strongest hypothesis and distinguished database-local health from ETL-host connectivity.

It used the network check failure at 02:55 AM and the connection timeout at 03:00 AM as supporting evidence.

It also acknowledged that the exact underlying network mechanism was not established.

**Strengths:**
- Correct diagnostic direction.
- Appropriate use of timestamps and health checks.
- Actionable network troubleshooting recommendations.
- Good uncertainty handling.

**Limitation:**
- Multiple network mechanisms were proposed without confirmation, although they were appropriately presented as possibilities.

**Ground Truth Alignment:** Strong.

### 3.2 INC-002 — Missing Records During Transformation

**Score: 24/25**

Claude correctly localized the discrepancy to the transformation stage.

The source produced 12,000 records, transformation produced 10,800, and the destination received the same 10,800 records.

It identified transformation filters and rejection logic as likely explanations while acknowledging that the exact rule was unknown.

**Strengths:**
- Correct pipeline-stage identification.
- Strong interpretation of record counts.
- Identification of missing rejected-record details.
- Practical investigation recommendations.

**Limitation:**
- The interpretation of the round 10% discrepancy as evidence of a systematic cause was not independently established.

**Ground Truth Alignment:** Strong.

### 3.3 INC-003 — Schema Mismatch

**Score: 24/25**

Claude correctly identified the unexpected `loyalty_tier` column as the immediate cause of schema validation failure.

It recommended confirming the source change and reviewing schema evolution policies before choosing a remediation path.

**Strengths:**
- Correct identification of the schema mismatch.
- Appropriate explanation of strict validation.
- Multiple remediation options.
- Good handling of uncertainty regarding the source change.

**Limitation:**
- Some explanations about uncoordinated releases and schema policies were speculative.

**Ground Truth Alignment:** Strong.

### 3.4 INC-004 — Duplicate Payment Records

**Score: 22/25**

Claude correctly identified duplicate `payment_id` values in the destination and recognized that the available evidence could not establish whether the duplicates originated in the source or during loading.

It recommended checking source duplicates, write mode, retry behaviour and watermark handling.

**Strengths:**
- Correct identification of the data quality issue.
- Recognition of the distinction between source and pipeline-generated duplicates.
- Useful investigation and remediation steps.
- Appropriate uncertainty handling.

**Limitations:**
- Claude asserted that the pipeline lacked uniqueness enforcement without sufficient supporting evidence.
- Several potential causes were discussed without confirmation.

**Ground Truth Alignment:** Good, with an unsupported assertion.

### 3.5 INC-005 — Missing S3 Input Object

**Score: 24/25**

Claude correctly identified the missing S3 input object as the immediate reason the transformation task failed.

It distinguished upstream task completion from successful delivery of the expected object.

**Strengths:**
- Correct identification of the missing dependency.
- Appropriate use of the NOT_FOUND error and retry information.
- Actionable S3 and pipeline handoff diagnostics.
- Explicit acknowledgement of missing upstream output-location information.

**Limitation:**
- Several possible mechanisms were proposed without confirmation.

**Ground Truth Alignment:** Strong.

## 4. Aggregate Performance

| Evaluation Dimension | Score | Maximum |
|---|---:|---:|
| Diagnostic Accuracy | 25 | 25 |
| Evidence Grounding | 24 | 25 |
| Hallucination Avoidance | 19 | 25 |
| Practical Usefulness | 25 | 25 |
| Uncertainty Handling | 25 | 25 |
| **Overall** | **118** | **125** |

**Overall Percentage: 94.4%**

**Performance Interpretation: Excellent**

## 5. Key Findings

The basic prompting experiment demonstrated that Claude could investigate a variety of synthetic data engineering incidents using a general-purpose prompt.

Key observations:

1. Claude consistently identified the primary failure area across all five scenarios.
2. It demonstrated strong reasoning using timestamps, record counts, error messages and health checks.
3. It generated actionable troubleshooting and preventive recommendations.
4. It generally distinguished confirmed observations from possible explanations.
5. Unsupported assertions and speculative explanations remained the primary area for improvement.

Hallucination avoidance received the lowest aggregate score, particularly in INC-004, where Claude asserted an unverified absence of uniqueness enforcement.

## 6. Limitations

- The experiment used five synthetic incident scenarios.
- Evaluation was conducted manually.
- Scoring involved qualitative judgement.
- The results do not establish performance across all production data engineering environments.
- The evaluation does not establish that Claude's recommendations were executed or independently verified in a live environment.

## 7. Conclusion

Experiment A established a baseline for Claude's ability to investigate synthetic data engineering incidents using basic prompting.

Claude demonstrated strong diagnostic reasoning, evidence interpretation, practical troubleshooting and uncertainty handling.

However, the evaluation also identified opportunities to improve the distinction between verified facts and plausible but unconfirmed explanations.

Experiment A achieved an overall score of 118/125 (94.4%) under the project's evaluation rubric.

The same evaluation methodology will be applied to Experiment B — Structured Prompting to assess whether more explicit prompt instructions improve response quality and reliability.