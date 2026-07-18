# Architecture Decisions

## Why a single LLM call?

To reduce latency and API costs.

---

## Why Google Sheets?

Simple persistence for the MVP.

Future versions will use PostgreSQL.

---

## Why human approval?

To avoid accidental responses.

---

## Why JSON validation?

To ensure workflow reliability.

---

## Why store before notification?

To avoid data loss if Telegram fails.

---

## Future Improvements

- PostgreSQL
- Redis Queue
- Multi-account support
- Multi-LLM fallback
- Dashboard
- RAG