# Installation Guide

## Requirements

- n8n
- Gmail account
- Gemini API Key
- Telegram Bot
- Google Sheets

---

## Credentials

Configure the following credentials inside n8n:

- Gmail OAuth2
- Google Sheets OAuth2
- Gemini API
- Telegram Bot

---

## Google Sheet

Create a sheet with the following columns:

- gmailId
- threadId
- from
- subject
- cleanBody
- category
- summary
- reply
- status
- notificationStatus
- createdAt

---

## Import

Import:

workflow.json

inside n8n.

---

## Configure

Replace:

- Sheet ID
- Telegram Chat ID
- Gemini API Key

with your own values.

---

## Activate

Enable the workflow.

Send a test email.

Verify:

- Google Sheet updated
- Telegram notification received
- Reply draft generated

---

## Troubleshooting

### JSON parsing

Check Gemini output.

### Google Sheets

Verify headers.

### Gmail

Verify OAuth permissions.

### Telegram

Verify Bot Token.