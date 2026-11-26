# Experiment A — Basic Prompting

## 1. Objective

Establish a baseline for evaluating Claude's ability to investigate data engineering incidents using a simple, general-purpose prompt.

The experiment aims to observe how Claude interprets incident information, identifies potential root causes, uses available evidence, and recommends troubleshooting steps without extensive prompting instructions.

## 2. Experimental Configuration

| Parameter | Configuration |
|---|---|
| Experiment ID | EXP-A |
| Experiment Name | Basic Prompting |
| AI Model | Claude Sonnet 5.5 |
| Effort Level | Medium |
| Prompt Type | Basic |
| Dataset | Synthetic Data Incident Dataset |
| Number of Incidents | 5 |
| Ground Truth | Not provided to Claude |
| Temperature | Default |
| Evaluation Method | Manual review against ground truth |

## 3. Prompt Reference

The experiment uses the reusable prompt documented in:

`prompts/basic_prompt.md`

## 4. Testing Procedure

1. Open a fresh Claude conversation for each incident.
2. Submit the basic incident triage prompt.
3. Insert the corresponding incident JSON as input.
4. Record Claude's complete response.
5. Preserve each response without editing its content.
6. Repeat the process for all five incident scenarios.
7. Conduct a systematic evaluation against the independent ground-truth references after collecting all responses.

## 5. Evaluation Focus

The response will be assessed based on:

- Accuracy
- Evidence usage
- Hallucination avoidance
- Practical usefulness
- Uncertainty handling

## 6. Experiment Scope

This experiment establishes the baseline for comparison with Experiment B — Structured Prompting.

The same incident inputs and model configuration will be used wherever possible to ensure a fair comparison.

## 7. Incident Execution Status

| Incident ID | Scenario | Response Status |
|---|---|---|
| INC-001 | Database connection timeout | Completed |
| INC-002 | Missing records during transformation | Completed |
| INC-003 | Schema mismatch | Completed |
| INC-004 | Duplicate payment records | Completed |
| INC-005 | Missing S3 input object | Completed |

## 8. Overall Experiment Status

Completed.

All five synthetic incident scenarios have been tested using the basic prompting approach.

The original Claude responses have been collected and preserved for subsequent evaluation against the independent ground-truth references.

The next stage involves defining the evaluation rubric and assessing the responses systematically before proceeding to Experiment B — Structured Prompting.