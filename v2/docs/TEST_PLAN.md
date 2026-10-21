# Verification plan and release gates

Run in a dedicated mailbox, private Telegram chat, and non-production ledger. Record n8n version and pass/fail for each test.

| ID | Scenario | Expected result |
|---|---|---|
| T01 | Ordinary email requesting a reply | Valid triage, evidence, pending ledger record; no email sent before approval |
| T02 | Empty body / HTML-only body | Safe normalization; uncertain classification, no invented facts |
| T03 | Same message delivered twice sequentially | Existing record blocks second intake |
| T04 | Malformed JSON from model | Validation error; no approval card or send |
| T05 | Confidence NaN, Infinity, string or out of range | Validation blocked |
| T06 | Unknown schema key, oversized draft, invalid category | Validation blocked |
| T07 | Prompt injection in email body | Treated as untrusted text; no policy change or autonomous action |
| T08 | Legal, financial, security or personal-data content | Risk flags visible; human approval required |
| T09 | Telegram callback from non-allowlisted actor | Rejected; ledger unchanged |
| T10 | Expired approval | Rejected; no send |
| T11 | Replayed approval callback | Rejected after state changes from pending |
| T12 | Reject callback | State becomes rejected; no Gmail reply node executed |
| T13 | Ledger unavailable | Fail closed; no approval/send |
| T14 | Gmail reply succeeds | Ledger moves to sent |
| T15 | Gmail accepts reply but workflow times out before ledger update | send_uncertain; inspect Gmail Sent before recovery |
| T16 | Telegram unavailable | Ledger record remains available; alert/recovery required |
| T17 | Very large body / long quoted thread | Analysis capped at 12,000 characters; original remains unchanged |
| T18 | Secret scan of files and git history | No credential values; revoke any exposed secret |
| T19 | Workflow restart while state is sending | Do not blindly resend; reconcile manually |
| T20 | Callback posted in wrong chat | Callback blocked; no draft sent |

## Known test limitations

The current Data Table design does not guarantee a race-safe unique key or atomic compare-and-set. T03 is only a sequential duplicate test; add a concurrent duplicate test and migrate to a transactional database before production use.

## Release criteria

- All critical tests pass on the actual target n8n version.
- No secrets appear in tracked files, exports, notifications or logs.
- No send path bypasses allowlist, pending-state, expiry and explicit approval checks.
- Ambiguous send results are reconciled before retry.
- Documentation matches imported node settings.
