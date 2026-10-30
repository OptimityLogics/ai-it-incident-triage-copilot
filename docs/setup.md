# Setup and verification notes

## Expected database objects

The workflow queries and writes the following PostgreSQL objects: `incidents`, `audit_log`, `failed_records`, `classifications`, `recommendations`, `runbooks`, `runbook_chunks`, and `past_incidents`. The retrieval query uses pgvector's `<=>` distance operator and expects an `embedding` vector column compatible with the configured embedding dimensions. This repository does not include DDL or seed data, so obtain the approved schema and runbook corpus from the project owner before deployment.

## Credentials

Configure credentials in n8n for PostgreSQL and the Gemini HTTP Header Auth nodes. Do not put API keys in the workflow JSON or this repository. Confirm the expected Gemini API key header name and value in your n8n credential setup.

## Suggested smoke tests

1. Valid incident with a title and description; expect a structured classification response.
2. Missing `id`; expect HTTP 422 and a validation error.
3. Missing both `description` and `logs`; expect HTTP 422.
4. Ambiguous incident; verify `needs_review` can be true.
5. No matching runbook context; verify the workflow flags review instead of fabricating a sourced plan.
6. Gemini failure or malformed model output; verify the rule-based classification fallback is recorded.

These are suggested verification cases, not a claim that this package has been executed against a live n8n/PostgreSQL environment.
