# AI IT Incident Triage & Root-Cause Copilot

An n8n workflow that accepts IT incidents through a webhook, validates the payload, classifies the incident with Gemini (or rule-based fallback), retrieves related runbook and historical-incident context from PostgreSQL using vector similarity, and returns a structured triage result with a suggested troubleshooting plan.

**Built by Optimity Logics**

![Optimity Logics — AI IT Incident Triage & Root-Cause Copilot](assets/banners/incident-triage-banner.png)

## Capabilities

- **Webhook intake:** receives incident reports over `POST /triage`.
- **Input validation:** checks required fields and timestamp format; invalid requests are recorded and returned with HTTP `422`.
- **Rule-assisted classification:** detects common database, storage, memory, network, certificate, deployment, authentication, and CPU symptoms.
- **AI classification:** uses Gemini to suggest incident type, severity, confidence, and evidence quotes.
- **Fallback and review flags:** falls back to rules when model output is unusable and marks uncertain or conflicting results for human review.
- **Retrieval-augmented troubleshooting:** creates an embedding and retrieves relevant runbook chunks and past incidents from PostgreSQL/pgvector.
- **Grounded action plan:** asks the model for a short plan using retrieved source excerpts; validates source titles and has an extractive fallback.
- **Auditability:** records incident status, classifications, recommendations, and processing events in PostgreSQL.

> AI classifications and remediation steps are decision support, not confirmed diagnoses. Review suggested actions before making production changes. The workflow prompt explicitly disallows automatic production changes.

## Repository structure

```text
.
├── README.md
├── workflow/
│   └── incident-triage-webhook.json
├── examples/
│   └── incident-request.json
├── docs/
│   └── setup.md
└── assets/
    └── banners/
        ├── incident-triage-banner.png
        └── incident-triage-end-banner.png
```

## Requirements

- An n8n instance that supports the nodes used in the exported workflow.
- PostgreSQL with the `pgvector` extension and the tables referenced by the workflow (`incidents`, `audit_log`, `failed_records`, `classifications`, `recommendations`, `runbooks`, `runbook_chunks`, and `past_incidents`).
- A Google Gemini API credential configured inside n8n for the HTTP Request nodes.
- A PostgreSQL credential configured inside n8n.
- Runbook and historical incident data embedded with the same embedding model and dimensionality used by this workflow.

**Database schema and seed data are not included in this package.** Configure these tables and load your runbooks/past incidents before expecting retrieval results.

## Quick start

1. Download or clone this repository.
2. In n8n, choose **Import from File** and import `workflow/incident-triage-webhook.json`.
3. Configure your own PostgreSQL and Gemini credentials on the corresponding nodes. Credential references were removed from this public-safe export and must be recreated in your n8n instance.
4. Verify the model names in the `Config` node against models available to your Gemini account.
5. Ensure PostgreSQL tables and vector data are ready; see [`docs/setup.md`](docs/setup.md).
6. Keep the workflow inactive while configuring it. Test using the n8n test webhook URL first, then activate it only after validation.
7. Send the example request below to the test webhook endpoint.

## Example request

```bash
curl -X POST 'http://localhost:5678/webhook-test/triage' \
  -H 'Content-Type: application/json' \
  --data @examples/incident-request.json
```

When the workflow is active, use your configured production webhook URL instead of `/webhook-test/`. The exact host and URL depend on your n8n deployment.

## Response behavior

- Invalid payloads return HTTP `422` with validation errors.
- Valid payloads return a JSON summary including incident type, severity, confidence, review status, plan method, number of plan steps, retrieved source titles, and best similarity score.
- The response can indicate `needs_review` when confidence is low, evidence is missing, classification sources disagree, or retrieval finds no sufficiently similar source.

## Configuration notes

The workflow's `Config` node currently defines model names, `LLM_MODE`, `CONF_THRESHOLD`, and `MIN_SCORE`. Review these settings before production use. `LLM_MODE` can be set to `mock` for rule-based classification testing, but retrieval still depends on a functioning database and embedding path in this workflow.

## Security

- Never commit API keys, database passwords, webhook secrets, or production incident data.
- Configure credentials through n8n's credential manager.
- Use network restrictions and authentication in front of the webhook when deploying beyond a local test environment.
- Review retention and access controls for incident descriptions, logs, and audit records.
- Treat retrieved runbooks and model outputs as untrusted input; verify recommendations before acting.

## Banner assets

Start and end cards are included under `assets/banners/` for use in the demo video. They are text-based Optimity Logics branded designs, not official logo artwork. Replace them with approved brand assets if required.

## License

Add the license approved by Optimity Logics before making this repository public.
