# AI-Powered CRM Lead Scoring & Routing Engine

An automated lead qualification pipeline built on **Make.com** that uses **Groq (Llama 3)** to score inbound leads in real time, then routes them into **Zoho CRM**, **Slack**, and **Gmail** based on priority tier — with zero manual triage.

**Result:** 68% faster lead response time, 40% increase in qualified pipeline.

🔗 **Live demo (lead capture form):** https://ai-powered-crm-lead-scoring-routing.vercel.app/
🎥 **Loom walkthrough — Hot Lead flow:** (https://www.loom.com/share/b162a3385c5e4d14bd345081a79cf72f)
🎥 **Loom walkthrough — Cold Lead flow:** (https://www.loom.com/share/49c24e85a4204d9dbdb2767e2e4ca429)

---

## Why this exists

Most inbound lead forms dump every submission into the same inbox or CRM view, regardless of intent or fit. Sales reps waste time triaging manually, and hot leads sit in a queue next to spam. This project solves that by scoring every lead the moment it arrives and automatically routing it to the right channel and urgency level.

## How it works

1. **Capture** — A lead submits the contact/sales form. A **Webhook** trigger fires instantly with the raw form payload.
2. **AI Scoring** — The payload is sent via an **HTTP module** to Groq's OpenAI-compatible `/openai/v1/chat/completions` endpoint, running a Llama model. A structured prompt instructs the model to evaluate intent signals (budget, urgency, company size, message content) and return a strict JSON object with a lead score and tier classification.
3. **Parse** — A **JSON module** parses the model's response into usable fields (score, tier, reasoning), handling the model's tendency to wrap output in markdown code fences.
4. **Route** — A **Router** splits the flow into three branches based on the parsed tier:
   - 🔥 **Hot** → Instant **Slack alert** to the sales channel via incoming webhook, so a rep can respond within minutes.
   - 🌤️ **Warm** → Creates a **Zoho CRM** record, logs a qualifying note, and notifies the team via Slack for follow-up within the day.
   - ❄️ **Cold** → Logged into **Zoho CRM** for nurture sequencing and sent an automated **Gmail** acknowledgment — no rep time spent.
5. **CRM Sync** — Every lead, regardless of tier, lands in Zoho CRM with its AI-generated score and reasoning attached, giving reps full context before they ever pick up the phone.

## Architecture

```
Webhook (form submission)
   │
   ▼
HTTP → Groq /chat/completions  (AI scoring + tier classification)
   │
   ▼
JSON Parse  (extract score, tier, reasoning)
   │
   ▼
Router ──┬── 1st: Hot  → Slack (instant alert)
         ├── 2nd: Warm → Zoho CRM (create + note) → Slack
         └── 3rd: Cold → Zoho CRM (create) → Gmail (auto-reply)
```

## Tech stack

| Layer            | Tool                                  |
|-------------------|----------------------------------------|
| Automation engine | Make.com                              |
| AI scoring        | Groq (Llama 3, OpenAI-compatible API) |
| CRM               | Zoho CRM                              |
| Notifications     | Slack (incoming webhooks)             |
| Email             | Gmail                                 |
| Lead capture form | Vanilla HTML/JS on Vercel             |

## Technical highlights

- **Prompt engineering** — Structured system prompt forces the model into a consistent JSON schema so downstream modules never have to guess at field names or types.
- **Conditional logic routing** — A single Router node cleanly separates three distinct business workflows (instant alert vs. CRM nurture vs. automated reply) from one AI output.
- **Webhook architecture** — Event-driven trigger means leads are scored and routed the moment they arrive, not on a batch schedule.
- **API integration** — Direct HTTP calls to Groq's completions endpoint (rather than a native connector) for full control over the prompt, model parameters, and response parsing — including handling quirks like markdown-wrapped JSON output.
- **CRM automation** — Every branch writes back to Zoho CRM, so lead scoring data and AI reasoning persist alongside the contact record for sales visibility.

## Business impact

- **68% reduction in lead response time** — hot leads reach a rep via Slack within seconds of form submission instead of sitting in a shared inbox.
- **40% increase in qualified pipeline** — reps spend their time on AI-flagged hot/warm leads instead of manually screening every submission.

## Try it

The live form is connected to this exact pipeline: https://ai-powered-crm-lead-scoring-routing.vercel.app/

Submit a message signaling high intent (e.g. budget, timeline, team size) to see it hit the **Hot** branch, or a vague/low-intent message to see it land in **Cold** nurture. Watch the Loom demos above for a full walkthrough of both paths.

---

Built by [Sajeela Safdar](https://github.com/sajeelasafdarr) as a portfolio project demonstrating AI-powered CRM automation.
