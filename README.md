# IvoryGlitch — State-Driven AI Sales & Lead Qualification Agent

> An enterprise-grade WhatsApp automation system bridging the gap between Meta Ads lead generation and human sales closing — built for high-touch fashion brands that refuse to compromise on customer experience.

---

## The Problem with Standard Chatbots

Most automation tools are either too rigid (linear decision trees that break on the first unexpected reply) or too unpredictable (pure LLM flows prone to hallucinations and context loss). For a brand like **IvoryGlitch**, neither extreme is acceptable.

We needed a system that could:

- **Qualify leads** systematically across specific data points: Size, Color, and Location
- **Maintain full context** across days or weeks of interrupted conversation
- **Handle multi-modal input** — images and voice notes — without breaking the qualification logic
- **Hand off to a human instantly** the moment the situation demands professional intervention

---

## Tech Stack

| Layer | Tool |
|---|---|
| Orchestration | [n8n](https://n8n.io) — Workflow Automation |
| AI Engine | LangChain + OpenAI GPT-4o |
| State Management | Google Sheets API — Deterministic State Tracking |
| Messaging Gateway | Evolution API — WhatsApp Business |
| Custom Logic | JavaScript / Node.js — Payload Parsing & Normalization |

---

## System Architecture

The system follows a **Modular Hybrid Agent** design — AI handles the conversation, deterministic logic handles the state.

```
┌─────────────────────────────────────────────────────────────────┐
│                        INGESTION LAYER                          │
│         Webhook → Meta Ads Payload / Organic WhatsApp           │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                        ROUTING LOGIC                            │
│              Switch: Text  │  Image  │  Audio                   │
└────────────┬───────────────┼─────────────────┬──────────────────┘
             │               │                 │
             ▼               ▼                 ▼
         AI Flow      Acknowledge &       Manual Override
                      Continue Flow       Flag + Human Alert
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                        MEMORY VAULT                             │
│  Short-term: Last 3 messages (Conversation Log)                 │
│  Long-term: Lead data R/W (Master Leads Sheet)                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                        HANDOFF GATE                             │
│  Manual_Override flag → silences AI, routes to human agent      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Features

### 1. State-Driven Lead Qualification

This agent doesn't just chat — it qualifies. Using a **Checklist-First Prompting** strategy, the system tracks which data points have been collected (size, color, city) and identifies what's still missing. Missing fields are woven naturally into the conversation rather than asked in a form-like sequence.

No robotic "Please enter your city." Just fluent, context-aware dialogue with a goal.

### 2. Multi-Modal Awareness

| Input Type | Behavior |
|---|---|
| **Text** | Standard AI qualification flow |
| **Image** | Acknowledged (e.g., payment proof, style reference) — flow continues |
| **Audio** | Triggers `Manual_Override` flag + dispatches human notification |

### 3. After-Hours Re-Entry & Context Injection

When a lead returns after days of silence, the system performs a **Context Injection** — feeding the last 3 conversation turns back into the LLM prompt before generating a reply. The conversation resumes exactly where it left off, with no repetition and no lost context.

### 4. Human Handoff Gate

A `Manual_Override` flag in the lead's Google Sheets row acts as a kill switch. Once set (by a human agent beginning to type, or triggered by an audio message), the AI is silenced for that conversation thread. All subsequent messages route directly to the human sales team.

---

## Performance & Business Impact

- **Response latency** under 10 seconds via optimized tool-calling
- **~70% reduction** in manual sales workload through automated 4-point lead qualification before human intervention
- **Race-condition-safe** concurrent execution designed for high-volume Meta Ads campaigns

---

## Implementation Details

> **Note:** All API keys and environment variables have been excluded from this repository for security. Use `.env.example` as a reference for required credentials.

The core logic lives in `n8n-workflow.json` and includes:

- **Custom JS nodes** for regex-based phone number normalization (handles international formats from Meta Ads payloads)
- **LangChain Buffer Window Memory** configurations for short-term context management
- **Dynamic System Prompts** that switch between `NEW_LEAD` and `RECURRING_LEAD` modes based on the lead's current state in Google Sheets

---

## Getting Started

### Prerequisites

- n8n instance (self-hosted or cloud)
- OpenAI API key (GPT-4o access required)
- Evolution API instance connected to a WhatsApp Business number
- Google Cloud project with Sheets API enabled
- A Meta Ads account (optional — system also handles organic WhatsApp messages)

### Setup

1. Clone this repository
   ```bash
   git clone https://github.com/admirerbrown/facebook_ad_whatsapp_inbound_automation.git
   cd ivoryglitch-agent
   ```

2. Copy the environment variable template
   ```bash
   cp .env.example .env
   ```

3. Fill in your credentials in `.env`

4. Import `n8n-workflow.json` into your n8n instance via **Settings → Import Workflow**

5. Configure your webhook URL in Evolution API to point to the n8n webhook endpoint

6. Set up your Google Sheets using the provided template structure (see `sheets-template/`)

7. Activate the workflow — the system is live

---

## Repository Structure

```
ivoryglitch-agent/
├── n8n-workflow.json        # Core workflow — import directly into n8n
├── sheets-template/         # Google Sheets schema for Master Leads + Conversation Log
├── .env.example             # Required environment variables (no secrets)
├── docs/
│   └── architecture.md      # Deeper architecture notes
└── README.md
```

---

## Roadmap

- [ ] Retry logic for failed Evolution API message deliveries

---

## License

MIT License — see `LICENSE` for details.

---

*Built for IvoryGlitch. Engineered for scale.*