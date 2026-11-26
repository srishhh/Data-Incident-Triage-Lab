# Synthetic Incident Dataset

This dataset contains five fictional data-engineering incidents for the AI Data Incident Triage — Prompting & Evaluation Lab.

## Structure

- `incidents/`: input records intended to be provided to the model.
- `ground_truth/`: independent evaluation references. Do not include these files in model prompts.

## Important evaluation note

Not every incident has a fully confirmed underlying cause. The ground-truth files distinguish:
- confirmed immediate failure or issue,
- strongest supported hypothesis,
- facts that are not established,
- expected model behaviour,
- unsupported claims to penalize.

This prevents the evaluation from rewarding confident guessing. In particular, INC-001 intentionally tests uncertainty handling.

All records are synthetic and contain no production data.
