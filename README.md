# NorthPeak HVAC — AI-Powered Lead Conversion System

An end-to-end lead automation pipeline built for NorthPeak Heating & Cooling, a US-based HVAC company receiving 40–60 leads per day. The system captures leads, qualifies them using AI, sends contextual responses, escalates urgent cases, and follows up automatically — all without human intervention.

---

## What It Does

- Captures leads instantly from a Tally form via webhook
- Qualifies each lead using Groq AI (urgency score, lead quality, sentiment, recommended action)
- Routes urgent leads (score ≥ 7) to the sales team via Telegram
- Sends AI-generated, contextual auto-response emails to normal leads
- Logs every lead to Google Sheets as a lightweight CRM
- Updates lead status automatically when a Calendly appointment is booked
- Sends follow-up emails at 1 hour and 24 hours if the lead hasn't booked

---

## System Architecture

```
Tally Form Submission
        ↓ (webhook)
n8n — Main Workflow
        ↓
Groq AI — Lead Qualification
(urgency score, lead quality, sentiment, summary)
        ↓
IF node — urgency ≥ 7?
   ├── YES → Telegram alert to sales team
   │          + Urgent email to lead ("we'll call you shortly")
   │          + Log to Google Sheets
   │
   └── NO  → AI-generated email to lead
              + Log to Google Sheets
              + Follow-up sequence (1hr, 24hr)

─────────────────────────────────────────

Calendly Booking (separate workflow)
        ↓
Update Google Sheets row → Status: "Appointment Booked"
(stops further follow-ups automatically)

─────────────────────────────────────────

Follow-up Workflows (schedule-triggered, every 15 min)
        ↓
Read Google Sheets — find leads older than 1hr / 24hr
with no follow-up sent and status ≠ "Appointment Booked"
        ↓
Send follow-up email → mark as sent in sheet
```

---

## Tech Stack

| Purpose | Tool | Why |
|---|---|---|
| Lead capture | Tally.so | Native webhook support on free tier |
| Automation engine | n8n Cloud | Execution-based billing, visual editor |
| AI qualification | Groq (llama3-70b-8192) | Free tier, sub-second responses |
| Email | Gmail via n8n | Simple OAuth connection |
| Internal alerts | Telegram Bot | Free, instant, 5-minute setup |
| Lead tracking | Google Sheets | Shared data layer between workflows |
| Appointment booking | Calendly | Free booking link, webhook on booking |

---

## Workflows

| File | Description |
|---|---|
| `main_workflow.json` | Core pipeline: form → AI → branch → email → sheets |
| `calendly_workflow.json` | Updates sheet status when appointment is booked |
| `followup_1hr.json` | Schedule-triggered 1 hour follow-up |
| `followup_24hr.json` | Schedule-triggered 24 hour follow-up |

To import into n8n: open n8n → New Workflow → top right menu → Import from file.

---

## Google Sheets Structure

**Sheet name:** `NorthPeak Lead Pipeline`

| Column | Description |
|---|---|
| Timestamp | When the form was submitted |
| Name | Lead's full name |
| Email | Lead's email address |
| Phone | Lead's phone number |
| Service | Type of HVAC service needed |
| Urgency Score | AI-assigned score 1-10 |
| Lead Quality | high / medium / low |
| Status | New Lead / Contacted / Qualified / Appointment Booked / Closed / Lost |
| Notes | AI-generated summary |
| Follow-up 1 Sent | Timestamp when 1hr follow-up was sent |
| Follow-up 2 Sent | Timestamp when 24hr follow-up was sent |

---

## Limitations

- **Sequential execution** — n8n free plan processes one lead at a time. High-volume spikes could delay processing for queued leads
- **No real SMS** — Twilio requires a paid account; email is used as the primary channel instead
- **Follow-up timing is approximate** — schedule runs every 15 minutes so a "1 hour follow-up" may arrive up to 1hr 15min after submission
- **Groq rate limits** — free tier could bottleneck at high volumes; a paid API key would be needed in production
- **Stateless AI** — the system has no memory of previous interactions with the same lead

---

## What I Would Improve With More Time

- Enable n8n queue mode (paid) for parallel lead processing
- Dedicated sentiment analysis using a fine-tuned NLP model
- Behaviour-based follow-ups (track email opens/clicks via SendGrid)
- AI voice calling via Vapi for urgent leads
- Replace Google Sheets with GoHighLevel CRM
- Live dashboard via Google Looker Studio
- Two-way SMS communication via Twilio
- Trained lead scoring model on historical conversion data

---

## Setup Instructions

1. Clone this repo
2. Create accounts on: n8n Cloud, Tally.so, Groq, Calendly
3. Import workflow JSON files into n8n
4. Create a Google Sheet with the column structure above
5. Connect credentials in n8n: Gmail, Google Sheets, Telegram Bot, Groq API key
6. Paste your n8n production webhook URL into Tally's webhook settings
7. Paste your Calendly booking link into both email prompts
8. Publish all workflows in n8n
9. Submit a test form and verify each node executes correctly

---

## Author

Built as part of an AI Automation Internship Assignment.  
GitHub: [your-username](https://github.com/your-username)
