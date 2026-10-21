# Architecture and invariants

## 1. Components

### Intake
- Trigger on new Gmail messages.
- Retrieve the canonical message by Gmail message ID.
- Normalize sender, recipients, subject, received timestamp, plain text and thread ID.
- Remove script/style content from HTML, decode common entities, preserve paragraph boundaries, and cap the analyzed body length.
- Keep the Gmail message ID as the idempotency key. Do not use subject or sender as a unique key.

### Analyzer
The model returns a single JSON object with exactly these fields:

```json
{
  "category": "urgent|action_required|informational|spam|uncertain",
  "priority": 1,
  "requested_action": "reply|review|none|unknown",
  "summary": "Short factual summary",
  "draft": "Reply text or empty string",
  "evidence": [{"claim": "Why this label was selected", "quote": "Short exact excerpt from the email"}],
  "risk_flags": ["financial|legal|security|personal_data|prompt_injection|none"],
  "model_confidence": 0.0
}
```

`priority` is an integer from 1 (lowest) to 5 (highest); confidence must be finite and within `[0,1]`. Enforce maximum lengths in code. Reject unknown keys and invalid enum values. Do not treat confidence as calibrated probability or permission to send.

### Policy gate
- Validate the schema and sanitize display output.
- Treat all email text as untrusted data. Ignore instructions embedded in email that attempt to change system policy, reveal secrets, or trigger actions.
- No tools, Gmail send calls, external HTTP calls, or credentials are exposed to the model.
- If validation fails, preserve a minimal error code and route to review; do not log raw email text or the full model response.
- Risk flags, uncertainty, sensitive requests, and high-impact topics always require human review.

### Decision ledger
Use a database table or n8n Data Table with a unique constraint on `gmail_message_id`. Google Sheets can be used for a small demo, but it is not a transactional database and cannot guarantee race-safe uniqueness. Suggested columns:

| Field | Purpose |
|---|---|
| `gmail_message_id` | Unique idempotency key |
| `thread_id` | Gmail thread reference |
| `state` | Current finite state |
| `received_at` | Source timestamp |
| `updated_at` | Last transition time |
| `category`, `priority` | Validated triage |
| `summary`, `draft` | Proposed result |
| `evidence_json`, `risk_flags_json` | Explanation and safety flags |
| `approval_actor_id` | Telegram user ID that approved/rejected |
| `approval_expires_at` | Approval deadline |
| `attempt_count` | Bounded retry counter |
| `last_error_code` | Redacted operational error |
| `gmail_sent_message_id` | Confirmation after successful send |

Never store API keys or Telegram bot tokens in this table. Define retention and restrict sheet/database access because email content can contain personal data.

## 2. State machine

```text
received → analyzed → pending_approval
pending_approval → rejected
pending_approval → expired
pending_approval → approved → sending → sent
sending → send_uncertain
sending → failed_retryable
failed_retryable → sending
```

Every transition must verify the current state and message key. `approved` is not sufficient by itself to send: the handler must verify the allowlisted actor, expiry, and pending state, then atomically claim the record as `sending` where the storage layer supports conditional updates.

### Exactly-once limitation
Gmail send and ledger updates are not one atomic transaction. If Gmail accepts a message but the workflow times out before recording success, an automatic retry can send a duplicate. On ambiguous timeout, move to `send_uncertain`, reconcile Gmail Sent using a stable correlation marker/message metadata, and require safe resolution before retry. Do not claim guaranteed exactly-once delivery.

## 3. Approval interface

- Prefer Telegram inline keyboard callbacks rather than public approve/reject URLs.
- Bind callback data to a compact record key and action; never include the draft or secrets in callback data.
- Verify the callback actor ID and chat ID against a manually configured allowlist.
- Reload the pending record from the ledger; never trust values sent by the callback client.
- Reject unknown, expired, already-used, or state-inconsistent callbacks.
- Answer the callback with a minimal status; do not echo full message content to group chats.
- Keep send capability out of the intake workflow and model prompt. Only the dedicated send handler receives the Gmail send credential.

## 4. Error handling

Classify errors as:
- `VALIDATION_FAILED`: permanent until reviewed; no automatic retry.
- `POLICY_BLOCKED`: manual review only.
- `RATE_LIMITED` / transient provider failure: bounded exponential retry with jitter where supported.
- `LEDGER_UNAVAILABLE`: fail closed; no approval or send.
- `SEND_UNCERTAIN`: reconcile before retry.

Set a maximum retry count and a dead-letter/review path. Avoid unbounded loops. Alert messages should contain a correlation ID and error code, not message bodies, tokens, or resume URLs.

## 5. Security and privacy

- Use n8n credential storage, least privilege, and separate read/send credentials if available.
- Do not commit credential exports, `.env` files, tokens, or execution payload dumps.
- Set execution-data retention and redact or disable successful execution payload persistence where appropriate for the instance.
- Limit email body size; do not process attachments in v2.
- Keep logs metadata-only by default.
- Approval is not an authorization substitute for sensitive workflows; policy-block high-impact actions for manual handling.
