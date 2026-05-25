# Universal AI Sales Agent

Telegram conversational agent for inbound lead qualification with native amoCRM integration. Free-form dialog — no buttons, no scripts. Production. Paid client deployment.

The agent collects qualification fields, scores the lead 0–100, answers questions via RAG over the client's knowledge base, escalates to a human on risk triggers, and writes the result back to amoCRM.

---

## What it does

- **Qualifies leads in free-form dialog** — collects request, timing, budget, contact; scores 0–100 against configurable rules
- **Answers via RAG** — retrieves from the client's knowledge base (services, prices, policies) and grounds every answer in source
- **Escalates on risk** — explicit triggers (price objection, churn signal, support emergency) hand the lead to a human immediately
- **Writes to amoCRM** — creates the lead card with custom fields auto-populated; assigns to the right pipeline stage
- **Monitors SLA** — alerts the manager when a hot lead has been idle for too long

---

## Lead scoring

| Score | Status | Action |
|---|---|---|
| 70–100 | Hot | Immediate handoff to manager |
| 40–69 | Warm | Queue + notification |
| 0–39 | Cold | Auto-nurturing flow |

Weights, thresholds, and triggers are config — not code.

---

## Architecture

```
Telegram
   │
   ▼
Aiogram 3 handlers
   │
   ▼
Conversation manager (FSM + LLM)
   │       │            │
   │       │            └──► RAG retriever ──► pgvector (knowledge base)
   │       │
   │       └──► Qualification scorer (rules + LLM extraction)
   │
   ▼
Decision router
   │       │            │
   │       │            └──► amoCRM API (lead card, custom fields, stage)
   │       │
   │       └──► Escalation queue ──► manager notification
   │
   └──► Audit log + SLA monitor
```

---

## Engineering decisions

**Free-form, not scripted.** Button-based bots die on the first unexpected request. The agent uses structured output to extract qualification fields from natural dialog while keeping the conversation human.

**RAG only — no model knowledge.** The agent never answers from training data about the client's business. If retrieval returns nothing, it says so and offers handoff. This is the only way to avoid hallucinated pricing or policies.

**Hard escalation triggers.** Some signals (price negotiation, complaint, emergency) bypass the agent entirely. The agent's job is to qualify, not to gamble with high-value moments.

**LLM fallback strategy.** On timeout or 5xx: graceful degradation to "I'll connect you with a manager" + queued retry. Never a stuck conversation.

**Config-driven customization.** Each client gets their own `qualification_rules.py`, `risk_triggers.py`, `tone.py`, and knowledge base — no code changes per deployment.

---

## Tech stack

- Python 3.13, Aiogram 3 (long polling)
- PostgreSQL 16 + pgvector
- OpenAI-compatible API
- Docker, docker-compose
- amoCRM REST API

---

## Quick start

```bash
git clone https://github.com/foggearthquake/universal-AI-agent-sells
cd universal-AI-agent-sells

cp .env.example .env   # BOT_TOKEN, OPENAI_API_KEY, DB, amoCRM tokens
docker compose up -d postgres
psql -U user -d db -f schema.sql

# Populate the knowledge base
# place .md/.txt files under execution/knowledge/

python -m bot
pytest
```

---

## Per-client customization

| File | Purpose |
|---|---|
| `config/qualification_rules.py` | Field weights, thresholds, scoring logic |
| `config/risk_triggers.py` | Escalation keywords and patterns |
| `config/tone.py` | Voice, formality, language |
| `execution/knowledge/` | Client knowledge base (chunked + embedded on startup) |

---

## Repo layout (DOE structure)

```
Directives/      # SOPs, qualification policy, escalation policy
Orchestration/   # prompts, conversation graph, scoring orchestration
Execution/       # code, integrations (amoCRM, DB), tests
```

---

**Author:** Ainur Gabdraupov — AI Architect / AI Engineer
[gabdra.pw](https://gabdra.pw) · [github.com/foggearthquake](https://github.com/foggearthquake)
