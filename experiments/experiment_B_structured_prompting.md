# Experiment B — Structured Prompting

## 1. Objective

Evaluate whether structured prompting improves Claude's ability to investigate synthetic data engineering incidents compared with the baseline established in Experiment A.

The experiment introduces explicit role definitions, investigation instructions, evidence requirements, uncertainty handling, constraints and standardized output formatting.

## 2. Experimental Configuration

| Parameter | Configuration |
|---|---|
| Experiment ID | EXP-B |
| Experiment Name | Structured Prompting |
| AI Model | Claude Sonnet 5.5 |
| Effort Level | Medium |
| Prompt Type | Structured |
| Dataset | Synthetic Data Incident Dataset |
| Number of Incidents | 5 |
| Ground Truth | Not provided to Claude |
| Temperature | Default |
| Evaluation Method | Same rubric used in Experiment A |

## 3. Prompt Reference

The experiment uses the structured prompt documented in:

`prompts/structured_prompt.md`

## 4. Testing Procedure

1. Open a fresh Claude conversation for each incident.
2. Submit the structured incident triage prompt.
3. Insert the corresponding incident JSON as input.
4. Record Claude's complete response.
5. Preserve each response without editing its content.
6. Repeat the process for all five incident scenarios.
7. Evaluate all responses against their independent ground-truth references.

## 5. Experiment Design

The structured prompt introduces:

- Role prompting.
- Context setting.
- Task decomposition.
- Evidence grounding.
- Explicit investigation constraints.
- Standardized output formatting.

The same five synthetic incident scenarios used in Experiment A will be used in this experiment.

## 6. Evaluation Focus

Responses will be evaluated using the same five dimensions established in `evaluation/evaluation_rubric.md`:

1. Diagnostic Accuracy.
2. Evidence Grounding.
3. Hallucination Avoidance.
4. Practical Usefulness.
5. Uncertainty Handling.

## 7. Incident Execution Status

| Incident ID | Scenario | Response Status |
|---|---|---|
| INC-001 | Database connection timeout | Pending |
| INC-002 | Missing records during transformation | Pending |
| INC-003 | Schema mismatch | Pending |
| INC-004 | Duplicate payment records | Pending |
| INC-005 | Missing S3 input object | Pending |

## 8. Overall Experiment Status

In Progress.

The structured prompt has been designed. All five incident scenarios will be tested before conducting the evaluation.

The results will subsequently be compared against Experiment A using the same evaluation methodology.