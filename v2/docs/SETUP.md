# Setup and rollout

## Prerequisites

- An n8n instance with a documented version recorded for the deployment.
- Gmail OAuth2 credentials with the minimum required read scope; grant send capability only to the dedicated send workflow.
- Google Gemini credentials or the chosen supported model credential configured in n8n's credential manager.
- Telegram bot and a private owner chat; record the numeric Telegram user ID and chat ID in instance configuration, not in source control.
- A private decision ledger. For production, prefer PostgreSQL or another transactional store with a unique constraint and conditional state transitions. Sheets is suitable only for a constrained demo.

## Configuration checklist

1. Import workflows into a non-production n8n project.
2. Assign credentials through the n8n UI. Do not paste keys into workflow JSON or expressions.
3. Create the ledger schema using the field contract in `ARCHITECTURE.md`.
4. Configure owner user/chat allowlists and approval expiry.
5. Set maximum analyzed body size and retention for execution data.
6. Disable workflow activation until the test plan passes.
7. Send test messages from a dedicated test mailbox; never test against real customer emails first.
8. Verify unauthorized Telegram accounts cannot approve, reject, or trigger sending.
9. Verify the approved draft is reloaded from the ledger immediately before send.
10. Record n8n version, node versions, credential scope, and test results in the deployment notes.

## Migration from the original workflow

- Keep the original workflow export as a reference; do not run both intake workflows against the same mailbox during testing.
- Remove the incomplete `Fill The DataBase` branch and the `resumeUrl`-based approval links from the old workflow.
- Do not rely on an `Update Row` operation unless a row is guaranteed to exist. Prefer insert-if-absent plus a unique key, or a transactional upsert.
- Rebuild field mappings from the ledger schema rather than copying the old mismatched sheet columns.
- Confirm the Gmail trigger's delivery semantics and test replay/retry behavior in the deployed n8n version.
- Keep the new workflow inactive until approval, rejection, duplicate, and timeout scenarios pass.

## Rollback

Disable the v2 workflows, preserve the ledger and execution metadata required for diagnosis, and return to manual inbox handling. Do not re-enable the original workflow until its invalid approval links and incomplete sheet branch have been disabled or repaired.
