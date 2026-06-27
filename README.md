# FitLife AI Sales System — Omnichannel Lead Qualifier with Voice Handoff

An end-to-end AI sales system that qualifies leads from WhatsApp and Instagram, scores them 1-10, routes them automatically, and triggers a real phone call within 30 seconds for hot leads — with zero human involvement.

Built with n8n, Gemini AI, Meta WhatsApp API, Google Sheets, and VAPI.

---

## Demo

> [Insert your demo video link here — YouTube or LinkedIn]

---

## System Architecture

```
Lead messages via WhatsApp or Instagram DM
 ↓
Normalize Input — detects channel, extracts message + language
 ↓
CRM Lookup — checks if lead already exists in Google Sheets
 ↓
Gemini AI Qualifier — scores lead 1-10, detects intent,
 budget, timeline, generates reply
 ↓
Route by Score:
 Score 1-6 → Nurture message sent back to lead
 Score 7-8 → Calendly booking link sent automatically
 Score 9-10 → VAPI triggers outbound call within 30 seconds
 + Hot lead email alert to coach
 ↓
Log to CRM — all leads logged to Google Sheets
 ↓
WhatsApp reply sent back to lead automatically
```

---

## Features

- **Omnichannel input** — accepts messages from WhatsApp (Meta API + Twilio) and Instagram DM (ManyChat webhook) through one unified entry point
- **AI lead scoring** — Gemini scores every lead 1-10 based on intent, budget mentions, and buying signals — not just keywords
- **Three-tier routing** — nurture, Calendly, or instant voice call based on score
- **30-second voice handoff** — hot leads (9-10) get a phone call triggered automatically via VAPI while they're still engaged
- **Automatic language detection** — detects English vs Urdu and responds in the same language
- **Google Sheets CRM** — every lead logged with score, intent, budget, timeline, source, and action taken
- **Hot lead email alert** — coach gets an instant email with full lead details when a 9-10 score comes in
- **Duplicate detection** — CRM lookup prevents logging the same lead twice
- **Graceful fallback** — if Gemini fails, sends a default nurture message instead of crashing

---

## Tech Stack

| Tool | Purpose |
|---|---|
| n8n (self-hosted) | Workflow orchestration |
| Gemini 2.5 Flash | Lead qualification + scoring + reply generation |
| Meta WhatsApp Cloud API | Receive + send WhatsApp messages |
| Google Sheets | Lead CRM + conversation log |
| VAPI | Outbound voice call for hot leads |
| Twilio | WhatsApp sandbox (alternative to Meta API) |
| Cloudflare Tunnel | Expose local n8n webhook publicly |
| SMTP | Hot lead email notifications |

---

## Repository Structure

```
fitlife-ai-sales-system/

 README.md

 n8n/
 fitlife_ai_sales_system.json # Complete n8n workflow

 prompts/
 gemini_qualifier_prompt.md # Full Gemini system prompt

 docs/
 architecture.png # n8n workflow screenshot
 crm_sample.png # Sample Google Sheets CRM output
```

---

## Setup Guide

### Prerequisites
- [n8n](https://n8n.io) — self-hosted via Docker
- [Meta Developer Account](https://developers.facebook.com) — for WhatsApp API
- [Gemini API key](https://aistudio.google.com/apikey) — free tier
- [VAPI account](https://vapi.ai) — for voice handoff (optional)
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps) — free, no account needed
- Google account — for Sheets

---

### Step 1 — Set Up Google Sheets CRM

Create a new Google Sheet called **"FitLife - Lead CRM"** with these exact headers in Row 1:

```
Timestamp | Lead Name | Phone Number | Source | Intent | 
Budget | Timeline | Lead Score | Status | Language | 
Conversation Log | Action Taken
```

---

### Step 2 — Set Up Meta WhatsApp API

1. Go to [developers.facebook.com](https://developers.facebook.com) → My Apps → Create App
2. Select **"Connect with customers through WhatsApp"**
3. Complete Basic Setup → claim your test number
4. Add your personal number as a verified recipient
5. Note down:
 - **Phone Number ID**
 - **WhatsApp Business Account ID**
 - **Access Token** (generate a permanent one in System Users)

---

### Step 3 — Import n8n Workflow

1. In n8n → New Workflow → Import from File
2. Upload `n8n/fitlife_ai_sales_system.json`
3. Replace these placeholders:

**In "Gemini Qualifier" Code node:**
```javascript
const GEMINI_API_KEY = 'PASTE-YOUR-GEMINI-API-KEY-HERE';
```

**In "Send WhatsApp Message" node:**
```
Phone Number ID: YOUR-PHONE-NUMBER-ID
Access Token: YOUR-META-ACCESS-TOKEN
```

**In "Send an Email" node:**
```
From: YOUR-EMAIL-HERE
To: YOUR-EMAIL-HERE
```

**In "Log to CRM" + "CRM Lookup" nodes:**
```
Sheet URL: YOUR-GOOGLE-SHEET-URL
```

4. Connect credentials:
 - Google Sheets → sign in with Google
 - SMTP → your email credentials
5. Toggle workflow to **Active**

---

### Step 4 — Configure Meta Webhook

1. In Meta App Dashboard → WhatsApp → Configuration
2. Set Callback URL:
 ```
 https://YOUR-CLOUDFLARE-URL.trycloudflare.com/webhook/fitlife-leads
 ```
3. Set Verify Token: `fitlife123` (or any string you choose)
4. Subscribe to: `messages`

---

### Step 5 — Run Cloudflare Tunnel

```bash
cloudflared tunnel --url http://localhost:5678
```

Copy the generated URL and update your Meta webhook callback URL.

---

### Step 6 — Test

Send a WhatsApp message to your test number. Try these scenarios:

**Low score (nurture):**
> "Hi, I want to get fit"

**Medium score (Calendly):**
> "I'm interested in your coaching program, what are the prices?"

**High score (voice call):**
> "I'm ready to start this week, my budget is $300/month"

After each message check:
- n8n executed green
- Correct action taken (nurture/Calendly/call)
- New row in Google Sheets
- WhatsApp reply received

---

## Google Sheets CRM Output Example

| Timestamp | Lead Name | Phone | Source | Intent | Budget | Timeline | Score | Status | Action Taken |
|---|---|---|---|---|---|---|---|---|---|
| 25/06/2026, 10:15 am | Ahmed | +923001234567 | whatsapp_meta | weight loss | unknown | unknown | 4 | nurture | nurture |
| 25/06/2026, 10:32 am | Sara | +923221234567 | whatsapp_meta | pricing inquiry | $200/month | next month | 7 | send_calendly | send_calendly |
| 25/06/2026, 11:05 am | Ali | +923451234567 | instagram | ready to buy | $300/month | this week | 9 | hot_lead_call | hot_lead_call |

---

## Customization

**Change the business niche:**
Update the Gemini system prompt in the "Gemini Qualifier" node — replace fitness coaching references with your client's industry.

**Adjust scoring thresholds:**
In the Gemini prompt, change the score ranges and action mappings to match your client's definition of a hot lead.

**Add more channels:**
The "Normalize Input" node already handles WhatsApp (Meta + Twilio), Instagram DM, and manual test format. Add more channel parsers to the if/else chain.

**Change Calendly link:**
In the "Edit Fields" node, replace the Calendly URL with your client's booking link.

---

## Cost Breakdown

| Item | Cost |
|---|---|
| Gemini API (free tier) | $0 |
| Meta WhatsApp (1000 msg/month) | $0 |
| n8n self-hosted | $0 |
| Google Sheets | $0 |
| Cloudflare Tunnel | $0 |
| VAPI voice call (per hot lead) | ~$0.15/call |
| **Total for demo** | **$0** |

---

## Before Uploading to GitHub

Replace these sensitive values in the workflow JSON:
```
AIzaSy... → PASTE-YOUR-GEMINI-API-KEY-HERE
your email → YOUR-EMAIL-HERE
Sheet ID → YOUR-SHEET-ID
instanceId → YOUR-INSTANCE-ID
credential IDs → YOUR-CREDENTIAL-ID
```

---

## Contact

Built by **Saleha** — AI automation & voice agent specialist

- LinkedIn: [your LinkedIn URL]
- Fiverr: [your Fiverr gig URL]
- Email: your.email@example.com

---

## License

MIT — free to use and modify for personal and commercial projects.
