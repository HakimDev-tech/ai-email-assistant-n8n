# Email Decision Engine v2

A safety-first n8n email triage system. It turns incoming messages into structured, evidence-backed recommendations and routes them through an explicit human decision gate. The model never authorizes side effects.

## Importable workflows

The JSON exports are in `workflows/` and remain inactive after import.

1. `00-bootstrap-ledger.json` — run manually once to create the `EmailDecisionLedger` Data Table.
2. `01-email-intake.json` — Gmail trigger → bounded normalization → duplicate lookup → Gemini triage → strict validation → ledger → Telegram approval card.
3. `02-approval-send.json` — Telegram callback → allowlist and expiry checks → ledger state validation → reject or reply to the original Gmail message → ledger update.

**Do not activate until credentials and placeholders are configured.** The workflow exports intentionally contain no credentials. Assign credentials in the n8n editor after import.

## Configure before first test

- Attach Gmail OAuth2 to the Gmail Trigger, Get Gmail Message, and Reply to Approved Email nodes. The reply node needs send permission.
- Attach a Gemini credential to the Gemini Chat Model node.
- Attach the same Telegram bot credential to Telegram Approval Card, Telegram Callback Trigger, Notify Rejection, Notify Sent, and Alert: Reconcile Before Retry.
- Replace the private Telegram chat ID placeholder in the intake workflow.
- Replace both Telegram user ID and private chat ID placeholders in the approval workflow's **Parse Callback + Fail Closed** code. Use numeric IDs as strings, not a username.
- In n8n, make sure only one Telegram Trigger is active for this bot; Telegram supports one webhook per bot.
- Run the bootstrap workflow once. Confirm the Data Table and all 21 columns exist.
- Import into a non-production project and send test emails from a test mailbox.

## Important limitations

- The Data Table workflow provides a practical starter ledger, but the exported design does not provide a transactional uniqueness constraint or a guaranteed atomic compare-and-set. Concurrent duplicate trigger executions can race. For stronger guarantees, migrate the ledger to PostgreSQL with a unique constraint on `gmail_message_id` and conditional state transitions.
- A Gmail reply and the ledger update cannot be committed atomically. If a reply request times out or fails ambiguously, the ledger becomes `send_uncertain`. Inspect Gmail Sent before attempting any manual recovery; do not simply re-approve.
- The current approval handler blocks non-allowlisted callbacks, expired approvals and non-pending states. It does not prove exactly-once email delivery.
- Email body is capped at 12,000 characters; attachments are not downloaded.
- No integration has been run against your n8n instance. Node versions, import compatibility, credential scopes, model output shape and actual Gmail/Telegram behavior must be tested on the target version.
- Do not store sensitive real email bodies in screenshots, issue reports or shared execution logs.

## Design principles

- Evidence over vibes: short source excerpts explain classification.
- Deterministic policy: model confidence never grants permission to send.
- Human control: every outbound reply requires an authorized approval callback.
- Privacy: bounded body, no attachment processing, and no raw resume URLs.
- Fail closed: malformed output, missing records, expired approvals and unauthorized callbacks block the action.

See `docs/ARCHITECTURE.md`, `docs/SETUP.md`, and `docs/TEST_PLAN.md`.
