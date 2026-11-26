# Basic Incident Triage Prompt

## Objective

Establish a baseline for evaluating Claude's ability to investigate data engineering incidents using a simple prompt.

## Prompt

Analyze the following data engineering incident.

Identify the possible root cause of the failure, explain what might have gone wrong, and suggest troubleshooting steps that could help resolve the issue.

Use the incident information and error logs provided below.

**Incident Data:**

[INSERT INCIDENT JSON HERE]

## Prompt Characteristics

This prompt provides a general task instruction without explicitly defining:

- An investigation methodology.
- Evidence citation requirements.
- Rules for handling uncertainty.
- Restrictions on unsupported assumptions.
- A standardized response format.

These elements will be introduced in the structured prompting experiment.

## Usage

This prompt will be reused across all five synthetic incident scenarios during Experiment A.