# AI Data Incident Triage — Project Objectives

## 1. Project Overview

AI Data Incident Triage is an experimental project exploring the application of Large Language Models (LLMs) in data engineering incident investigation and troubleshooting.

The project focuses on evaluating how effectively AI models can interpret synthetic data pipeline incidents, analyze error logs, identify potential root causes, and recommend actionable troubleshooting steps.

Through structured prompting and systematic evaluation, this lab aims to understand the capabilities, limitations, and reliability of AI-assisted incident triage.

## 2. Problem Statement

Modern data engineering environments involve complex ETL/ELT pipelines, database integrations, scheduled jobs, and data processing workflows.

When these systems fail, engineers must investigate logs, identify failure points, determine potential root causes, and recommend corrective actions.

Although LLMs can assist with interpreting technical information, their responses may contain inaccurate assumptions, unsupported conclusions, or fabricated details.

This project investigates whether structured prompting and evaluation techniques can improve the reliability and practical usefulness of AI-generated incident analysis.

## 3. Project Objectives

The primary objectives are:

- Explore LLM capabilities in interpreting data engineering incidents.
- Design synthetic datasets representing common data pipeline failures.
- Compare basic prompting against structured prompting techniques.
- Evaluate AI-generated responses for accuracy, evidence-based reasoning, and practical usefulness.
- Identify hallucinations, unsupported assumptions, and limitations.
- Develop a repeatable evaluation methodology for AI-assisted incident triage.
- Explore opportunities for automating response evaluation using Python.

## 4. Project Scope

The initial experimental scope includes five incident categories:

1. Database connection timeout.
2. Missing records and data loss.
3. Schema mismatch.
4. Duplicate records.
5. ETL pipeline execution failure.

The experiments will use synthetic incident descriptions, logs, and expected root causes.

The project will compare different prompting approaches and evaluate their responses against predefined evaluation criteria.

## 5. Evaluation Criteria

AI-generated responses will be assessed using five criteria:

| Criterion | Description |
|---|---|
| Accuracy | Correct identification of the underlying issue. |
| Evidence Usage | Reliance on information provided in the incident data. |
| Hallucination Avoidance | Avoidance of fabricated details and unsupported claims. |
| Practical Usefulness | Quality and relevance of troubleshooting recommendations. |
| Uncertainty Handling | Ability to acknowledge missing information and uncertainty. |

## 6. Project Limitations

- Experiments will initially use synthetic incident data rather than production infrastructure.
- AI-generated root causes will be treated as hypotheses, not verified diagnoses.
- The initial lab will not execute corrective actions on real systems.
- Evaluation results will reflect the tested scenarios and model configurations, not universal model performance.

## 7. Expected Outcomes

By completing this project, the expected outcomes are:

- A documented collection of synthetic data engineering incidents.
- A collection of basic and structured prompts.
- Comparative experiment results.
- A reusable evaluation rubric.
- Python-based evaluation utilities.
- Documented observations regarding LLM reliability and limitations.

## 8. Success Criteria

The project will be considered successful when:

- All five synthetic incident scenarios have been documented.
- Both prompting approaches have been tested.
- Responses have been evaluated using the predefined criteria.
- Experimental findings and limitations have been documented.
- A reproducible evaluation workflow has been developed.
