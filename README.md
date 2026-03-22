# Email Automation Agent

An intelligent, fully automated email management system built with n8n. Handles inbound email classification, auto-replies, GDPR compliance, follow-up scheduling, and daily reporting — all without human intervention.

## What It Does

This workflow monitors an email inbox 24/7 and automatically:

- **Classifies** every incoming email using OpenAI (GPT) into categories: lead, support, billing, internal, or other
- **Routes** emails by intent: normal reply, unsubscribe request, or GDPR data request
- **Auto-replies** with the appropriate response based on category
- **Manages GDPR compliance** — unsubscribe requests are instantly processed, contacts marked as opted-out, and confirmation sent
- **Alerts via Telegram** for high-priority emails and GDPR data requests requiring manual action
- **Schedules follow-ups** daily at 10:00 for conversations with no reply in 3+ days
- **Sends a daily report** to Telegram at 20:00 with full activity summary

## Architecture

```
IMAP Trigger → Parse Email → Upsert Contact (Supabase)
    → AI Classify (OpenAI GPT) → Route by Intent
        ├── Unsubscribe → Mark Opted Out → Confirm + Notify Telegram
        ├── GDPR Request → Log + Acknowledge + Alert Telegram
        └── Normal → Route by Category
            ├── Support → Auto-reply
            ├── Billing → Auto-reply
            └── Lead → Check Consent → Send Reply

Schedule (10:00) → Find Stale Conversations → Queue Follow-ups
Schedule (every 10min) → Process Queue → Send/Block based on consent
Schedule (20:00) → Generate Daily Report → Send to Telegram
```

## Tech Stack

- **n8n** — workflow orchestration
- **OpenAI GPT** — email classification and intent detection
- **Supabase** — contact management, conversation tracking, event logging, outbound queue
- **IMAP** — email inbox monitoring
- **Telegram Bot** — real-time alerts and daily reports
- **SMTP** — automated email sending

## Key Features

- **GDPR-compliant** — full unsubscribe handling with consent tracking and data request acknowledgment
- **Consent-gated sending** — emails are never sent to opted-out contacts, queue items are marked as blocked
- **Priority alerting** — high-priority emails trigger instant Telegram notifications
- **Full audit trail** — every action logged to Supabase events_log table
- **Queue-based outbound** — follow-ups processed via a reliable queue with retry safety

## Setup

1. Import the workflow JSON into your n8n instance
2. Configure the **Workflow Configuration** node with your:
   - Telegram Chat ID
   - Company name
   - Support email address
3. Connect credentials:
   - IMAP account (your email inbox)
   - OpenAI API key
   - Supabase project URL and key
   - Telegram Bot token
4. Set up Supabase tables: `contacts`, `conversations`, `events_log`, `outbound_queue`
5. Activate the workflow

## Database Schema (Supabase)

| Table | Key Fields |
|-------|-----------|
| contacts | id, email, name, consent_status, last_interaction |
| conversations | id, contact_id, thread_id, category, priority, status, last_inbound, last_outbound |
| events_log | id, contact_id, action, details, ts |
| outbound_queue | id, contact_id, type, subject, body, scheduled_at, status, block_reason |

---

Built by [Shehroz Khan](https://www.linkedin.com/in/shehroz-khan-b91716197/) · Fullstack Automation Engineer
