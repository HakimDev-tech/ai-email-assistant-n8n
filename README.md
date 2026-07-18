# AI Email Assistant

A production-oriented AI-powered email automation workflow built with n8n.

## Overview

AI Email Assistant automates repetitive email processing while keeping humans in control.

The workflow:

- Detects new Gmail emails
- Cleans HTML content
- Uses Gemini AI for analysis
- Classifies emails
- Generates summaries
- Creates professional reply drafts
- Sends Telegram notifications
- Requires human approval before sending
- Stores processing history in Google Sheets

No email is ever sent automatically.

---

## Features

- Gmail Trigger
- HTML preprocessing
- AI email classification
- AI summarization
- AI reply generation
- Telegram approval workflow
- Google Sheets persistence
- Error handling
- Retry strategy
- Human-in-the-loop validation

---

## Architecture

```text
Gmail Trigger
      ↓
HTML Cleanup
      ↓
Google Sheets
      ↓
Gemini AI
      ↓
JSON Validation
      ↓
Google Sheets Update
      ↓
Telegram Notification
      ↓
Human Approval
      ↓
Gmail Send
```

---

## Tech Stack

- n8n
- Gemini API
- Gmail API
- Google Sheets API
- Telegram Bot API

---

## Workflow Highlights

- Single LLM call
- JSON-only responses
- Human approval required
- Error recovery
- Logging
- Retry mechanism
- Idempotent processing using Gmail Message ID

---

## Future Improvements

- PostgreSQL
- Redis Queue
- Multi-account support
- Attachment analysis
- Calendar integration
- Multi-LLM fallback
- Dashboard

---

## Installation

1. Clone the repository.

2. Import the workflow into n8n.

3. Configure credentials:

- Gmail
- Gemini
- Google Sheets
- Telegram

4. Update your Google Sheet IDs.

5. Activate the workflow.

---

## Project Goals

This project was built to practice production-grade AI workflow design using n8n.

The focus is on:

- maintainability
- robustness
- scalability
- AI integration
- automation architecture

---

## License

MIT
