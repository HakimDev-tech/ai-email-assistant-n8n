# Email Decision Engine v2

A safety-first n8n email triage system. It turns incoming messages into structured, evidence-backed recommendations and routes them through an explicit human decision gate. The model never authorizes side effects.

## Design principles

- **Evidence over vibes:** every material classification should include short supporting excerpts from the source email.
- **Deterministic policy:** code and workflow rules decide whether a draft can be offered or sent; model confidence alone never grants permission.
- **Human control:** the initial release is draft-and-approve only. No automatic sending based on model output.
- **Idempotency:** Gmail message ID is the unique processing key; retries must not create a second decision or send.
- **Fail closed:** malformed model output, missing fields, failed state lookups, expired approval, or identity mismatch must block sending.
- **Data minimization:** do not send attachments or unnecessary thread history to the model; avoid logging message bodies and secrets.

## Scope of this v2 branch

This branch establishes the implementation contract, data model, test gates, and migration procedure. It is deliberately separate from `main`. Do not activate a workflow until the instance-specific credentials, table schema, Telegram identity allowlist, and integration tests are configured.

## Workflow boundaries

1. **Inbox intake:** capture message metadata and normalized text; compute a stable message key; reject duplicates before analysis.
2. **Analysis:** request schema-constrained triage and draft generation. Email text is untrusted input, not instructions.
3. **Policy gate:** validate the exact output schema, field lengths, category enums, evidence spans, and finite numeric scores. Invalid output is quarantined for review.
4. **Decision ledger:** persist `received → analyzed → pending_approval → approved/rejected/expired → sent/failed`. Every transition is checked against the current state.
5. **Approval handler:** accept callbacks only from the configured Telegram user/chat; validate message key and pending state; reject reused/expired approvals.
6. **Send handler:** reload the approved draft from the ledger, send only that persisted draft, and record the result. Never trust a draft supplied in a callback URL or payload.

## Differentiation

- **Evidence-linked triage:** short source excerpts explain urgency and requested action.
- **Decision receipts:** record why a message was classified, what was proposed, who approved it, and what action actually succeeded.
- **Risk-aware routing:** financial, legal, account-security, sensitive-personal-data, and ambiguous messages default to manual review.
- **Failure-aware states:** retryable infrastructure errors are distinguished from permanent validation/policy blocks.
- **Privacy-aware prompts:** bounded text, no attachments by default, and no raw message bodies in Telegram alerts or routine logs.

## Required configuration

Create credentials inside n8n; never commit tokens or credential exports. Use separate least-privilege credentials for Gmail read and send where practical. Configure a private decision ledger and a Telegram owner allowlist. Exact node settings depend on the deployed n8n version and installed node versions; validate imported workflow JSON on the target instance before activation.

See `docs/ARCHITECTURE.md`, `docs/SETUP.md`, and `docs/TEST_PLAN.md`.

## Release gate

The system is not production-ready until all critical tests pass: duplicate delivery, malformed AI response, prompt injection, unauthorized callback, expired/reused approval, Gmail send timeout, and ledger failure. An approval send that times out after Gmail accepted the message requires reconciliation before retry; Gmail send is not a transactional exactly-once operation.
