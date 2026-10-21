# Setup and rollout

## Import order

1. Import `workflows/00-bootstrap-ledger.json`.
2. Run it manually once. Confirm that `EmailDecisionLedger` exists with all 21 columns.
3. Import `workflows/01-email-intake.json`.
4. Import `workflows/02-approval-send.json`.
5. Assign credentials and replace placeholders below. Keep all three workflows inactive until testing is complete.

## Credentials and placeholders

- `01-email-intake.json`: assign Gmail OAuth2 to **Gmail Trigger** and **Get Gmail Message**; assign Gemini to **Gemini Chat Model**; assign Telegram to **Telegram Approval Card**. Replace `REPLACE_WITH_PRIVATE_TELEGRAM_CHAT_ID` with the numeric ID of the private owner chat.
- `02-approval-send.json`: assign Telegram to **Telegram Callback Trigger**, **Notify Rejection**, **Notify Sent**, and **Alert: Reconcile Before Retry**. Assign Gmail OAuth2 with send/reply permission to **Reply to Approved Email**.
- In **Parse Callback + Fail Closed**, replace `REPLACE_WITH_TELEGRAM_USER_ID` with your numeric Telegram user ID and `REPLACE_WITH_PRIVATE_TELEGRAM_CHAT_ID` with your private chat ID. Both must match the callback actor and callback message chat exactly.
- Keep the same Telegram bot credential across these nodes. Only one Telegram Trigger can register a webhook for a bot at a time; this design uses one trigger in the approval workflow.
- Set credentials using n8n's credential manager. Never paste API keys into workflow expressions or commit credential exports.

## Controlled activation

1. Import into a test n8n project and record the n8n version.
2. Configure credentials and placeholders.
3. Run the bootstrap workflow and verify the schema.
4. Test the intake with a dedicated test mailbox while the approval workflow is inactive; verify a pending record is saved and a Telegram card is sent.
5. Activate the approval workflow, then the intake workflow. Send a fresh test email.
6. Test rejection first: ledger should move to `rejected`, with no reply sent.
7. Test approval using a non-sensitive test email. Confirm the original message is replied to and the ledger moves to `sent`.
8. Test unauthorized user, wrong chat, expired approval, duplicate callback, malformed model output, ledger outage, Telegram outage and ambiguous Gmail timeout.
9. Confirm the workflow remains inactive until these checks pass.

## Rollback

Deactivate the intake and approval workflows, preserve the ledger and necessary redacted execution metadata, and return to manual inbox handling. Do not reactivate the original workflow while its incomplete approval links or blank Sheets branch remain.
