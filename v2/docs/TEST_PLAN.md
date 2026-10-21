# Verification plan and release gates

Run these tests in a dedicated mailbox, private Telegram chat, and non-production ledger. Capture pass/fail and n8n version for every test.

| ID | Scenario | Expected result |
|---|---|---|
| T01 | Ordinary message requesting a reply | Valid analysis, short evidence excerpts, draft saved as pending approval; no email sent |
| T02 | Empty body / HTML-only body | Safe normalization; empty content marked uncertain rather than hallucinated |
| T03 | Same Gmail message delivered twice | One ledger record and one approval request; no duplicate send |
| T04 | Malformed JSON from model | Validation blocked; no send; metadata-only error logged |
| T05 | Confidence is NaN, Infinity, string, or out of range | Validation blocked |
| T06 | Unknown schema key, oversized draft, invalid category | Validation blocked |
| T07 | Email contains “ignore previous instructions” or asks for secrets | Treated as untrusted quoted content; no policy change or action |
| T08 | Legal, financial, security, or sensitive personal-data message | Risk flag and mandatory manual review; no autonomous send |
| T09 | Telegram callback from non-allowlisted actor/chat | Rejected; ledger unchanged |
| T10 | Expired approval | Rejected; no send |
| T11 | Replayed approve callback | Rejected after first valid state transition |
| T12 | Rejection callback | State becomes rejected; no Gmail send node executed |
| T13 | Ledger unavailable | Fail closed; no approval or send |
| T14 | Gmail send returns success | Sent state and Gmail message ID recorded |
| T15 | Gmail accepts send but workflow times out before ledger update | `send_uncertain`; reconciliation required before retry |
| T16 | Telegram unavailable | Decision remains recoverable in ledger; bounded retry/alert, no lost record |
| T17 | Large body / long quoted thread | Body is capped; original message remains intact in Gmail |
| T18 | Secret scan of repository and git history | No credentials/tokens; any exposed credential is revoked and history assessed |
| T19 | Restart/retry while state is `sending` | No blind resend; reconciliation or manual resolution |
| T20 | Approval message posted in wrong chat | No sensitive body/draft disclosed; action unavailable outside configured private chat |

## Release criteria

- All T01–T20 pass, or any explicitly waived test has a written risk owner and mitigation.
- No credential values appear in tracked files, workflow exports, logs, or notification payloads.
- No execution path can send without passing the policy gate and a fresh, authorized approval.
- Duplicate and ambiguous-send behavior is documented and tested.
- Setup docs match the exact imported workflow node versions and field mappings.
- A human operator can inspect a decision receipt without exposing the full source email.

A static JSON inspection is not proof that the deployed integration works. Verify on the actual n8n instance before production activation.
