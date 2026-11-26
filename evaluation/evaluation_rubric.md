# AI Data Incident Triage — Evaluation Rubric

## 1. Purpose

This document defines the evaluation methodology used to assess Claude's ability to investigate synthetic data engineering incidents.

The rubric establishes consistent evaluation criteria for measuring the accuracy, reliability and practical usefulness of AI-generated incident analysis.

It will be applied to both:

- Experiment A — Basic Prompting
- Experiment B — Structured Prompting

The same evaluation criteria and scoring methodology will be maintained across both experiments to ensure a fair comparison.

## 2. Evaluation Criteria

Each incident response will be evaluated using five dimensions.

### 2.1 Diagnostic Accuracy

**Objective:** Evaluate whether Claude correctly identifies the failure area and establishes a reasonable diagnosis based on the available incident information.

Assessment considerations:

- Correct identification of the affected pipeline stage or component.
- Alignment between the proposed root cause and the incident evidence.
- Ability to distinguish the actual failure area from unrelated components.
- Avoidance of unsupported definitive root-cause conclusions.

### 2.2 Evidence Grounding

**Objective:** Evaluate whether Claude supports its analysis using the information available in the incident input.

Assessment considerations:

- References to relevant logs, timestamps and error messages.
- Correct interpretation of record counts, metrics and health checks.
- Logical connections between observed evidence and proposed explanations.
- Identification of missing evidence required for further investigation.

### 2.3 Hallucination Avoidance

**Objective:** Evaluate whether Claude avoids inventing information or presenting unsupported assumptions as established facts.

Assessment considerations:

- No fabricated logs, metrics, configuration changes or infrastructure events.
- No unsupported claims about the underlying environment.
- Clear distinction between observed facts and possible explanations.
- Appropriate handling of information that is unavailable in the incident input.

### 2.4 Practical Usefulness

**Objective:** Evaluate whether Claude provides actionable recommendations that help an engineer investigate and resolve the incident.

Assessment considerations:

- Relevance of troubleshooting steps to the identified problem.
- Specificity of diagnostic commands and investigation techniques.
- Logical ordering and prioritization of troubleshooting actions.
- Safety and practicality of remediation recommendations.
- Inclusion of preventive measures where appropriate.

### 2.5 Uncertainty Handling

**Objective:** Evaluate whether Claude recognizes the limitations of the available information and communicates uncertainty appropriately.

Assessment considerations:

- Identification of missing information.
- Distinction between confirmed findings and hypotheses.
- Appropriate confidence assessment.
- Avoidance of premature conclusions.
- Recommendations for obtaining additional evidence before confirming a root cause.

## 3. Scoring Methodology

Each evaluation dimension will receive a score from 1 to 5.

| Score | Rating | Description |
|---|---|---|
| 1 | Poor | Major deficiencies; fails to meet the criterion. |
| 2 | Weak | Significant gaps; meets only a limited part of the criterion. |
| 3 | Satisfactory | Partially meets expectations but has identifiable shortcomings. |
| 4 | Good | Meets expectations with minor shortcomings. |
| 5 | Excellent | Fully meets expectations with strong supporting reasoning. |

Scores must be supported by observations from the actual Claude response and its corresponding ground-truth reference.

## 4. Evaluation Score Calculation

All five evaluation dimensions carry equal weight.

**Maximum score per incident: 25 points.**

Total score is calculated as:

Total Score = Diagnostic Accuracy + Evidence Grounding + Hallucination Avoidance + Practical Usefulness + Uncertainty Handling

The percentage score is calculated as:

Percentage Score = (Total Score / 25) × 100

For each experiment:

- Five incidents will be evaluated.
- Maximum experiment score: 125 points.
- Overall experiment percentage will be calculated using the combined scores.

## 5. Performance Interpretation

The following bands will be used to interpret the overall percentage score.

| Percentage | Interpretation |
|---|---|
| 90–100% | Excellent |
| 75–89% | Good |
| 60–74% | Satisfactory |
| 40–59% | Weak |
| Below 40% | Poor |

These bands are descriptive interpretations for this project and do not represent externally validated industry benchmarks.

## 6. Evaluation Procedure

Each incident response will be evaluated through the following process:

1. Retrieve the original Claude response from the corresponding experiment output.
2. Retrieve the independent ground-truth reference for that incident.
3. Compare Claude's findings against the ground truth and the original incident input.
4. Assess each of the five evaluation dimensions.
5. Assign a score from 1 to 5 for each dimension.
6. Document evidence and reasoning supporting each score.
7. Calculate the incident's total and percentage score.
8. Repeat the process for all five incidents.

The ground-truth references will not be provided to Claude during either experiment.

## 7. Comparative Evaluation

After evaluating all five incidents in both experiments, the results will be compared to determine how structured prompting influences Claude's performance.

The comparison will consider:

- Overall evaluation scores.
- Performance across individual evaluation dimensions.
- Improvements or regressions between experiments.
- Changes in evidence grounding and hallucination avoidance.
- Differences in diagnostic accuracy and practical usefulness.
- Consistency across different incident types.

## 8. Evaluation Principles

The following principles will be maintained throughout the project:

1. **Consistency:** The same evaluation rubric will be applied to both experiments.
2. **Evidence-based scoring:** Scores must be justified using actual response content and ground-truth references.
3. **Independence:** Ground-truth information will remain separate from Claude's input.
4. **Objectivity:** A longer or more detailed response will not automatically receive a higher score.
5. **Transparency:** Unsupported claims, missing information and incorrect conclusions will be explicitly documented.
6. **Reproducibility:** Evaluation results should be understandable and repeatable using the documented inputs and scoring methodology.

## 9. Scope and Limitations

This evaluation is based on five synthetic data engineering incident scenarios and manual assessment of AI-generated responses.

The results provide an experimental comparison of two prompting approaches within the scope of this project.

They should not be interpreted as a comprehensive benchmark of Claude's performance across all production data engineering incidents.