# Pipeline Architecture
**Credit Risk RAG Agent — High-level design**

---

## What the pipeline does

Takes a loan application → scores it → checks for fraud → retrieves policy context via RAG → composes a personalised email → sends it automatically. One-shot, fully automated, with a human escalation path for suspicious cases.

---

## Architecture overview

```
Loan application data
        │
        ▼
┌───────────────────────┐
│   Existing notebook   │
│  ┌─────────────────┐  │
│  │ Credit risk ML  │  │   ← your existing model
│  └────────┬────────┘  │
│  ┌─────────────────┐  │
│  │ OpenAI loan     │  │   ← categorizes loan purpose
│  │ categorizer     │  │
│  └────────┬────────┘  │
└───────────┼───────────┘
            │
            ▼
      ┌─────────────┐
      │  Decision   │  Approve / Reject / Suspicious
      └──────┬──────┘
    ┌────────┼─────────┐
    ▼        ▼         ▼
Approved  Rejected  Suspicious
    │        │         │
    │        │         ▼
    │        │   ┌──────────────┐
    │        │   │ Human review │  ← manual approve/reject
    │        │   └──────┬───────┘
    ▼        ▼          │
┌────────┐ ┌─────────┐  │
│Approval│ │Rejection│◄─┘
│ agent  │ │  agent  │
└───┬────┘ └────┬────┘
    │            │
    └─────┬──────┘
          ▼
  ┌───────────────────┐
  │  Policy vector    │  ← ChromaDB (local)
  │  store (RAG)      │     interest rates, offers,
  └────────┬──────────┘     legal disclaimers
           │
           ▼
  ┌───────────────────┐
  │  LLM email        │  ← OpenAI GPT-4o-mini
  │  composer         │     grounded in RAG context
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │  Gmail API sender │
  └────────┬──────────┘
           │
           ▼
    Applicant inbox
    (one-shot email)
```

---

## Layer breakdown

### Layer 1 — Input
Raw loan application data (CSV or database row). Contains applicant features, financials, and loan request details.

### Layer 2 — Existing notebook (your ML work)
Two things happen here, both already built:
- **Credit risk model** scores the application and produces an approve/reject decision with a confidence probability
- **OpenAI categorizer** labels the loan purpose (personal, home improvement, debt consolidation, etc.)

No changes needed to your existing notebook — this layer just feeds into the new pipeline.

### Layer 3 — Decision + fraud gate
The decision branches into three paths:

| Path | Condition | Next step |
|------|-----------|-----------|
| Approved | Score above threshold, no flags | Approval agent |
| Rejected | Score below threshold, no flags | Rejection agent |
| Suspicious | Any fraud flag triggered | Human reviewer |

The fraud gate runs before any agent fires. If it raises a flag, the application is held — no email is sent until a human resolves it.

### Layer 4 — Fraud / anomaly detector
A rule-based checker (`fraud_check.py`) that returns one of three statuses:

- `CLEAN` → pass to agent
- `REVIEW` → hold, alert human
- `AUTO_REJECT` → hard reject, no email (extreme cases)

**What triggers REVIEW:**
- Model confidence score in grey zone (e.g. probability 0.40–0.60)
- Loan amount > 10× annual income
- Implausible field values (negative age, employment longer than age, etc.)
- Same email on multiple recent applications with different names
- OpenAI categorizer returned `unknown` or very low confidence

### Layer 5 — Agent layer
Two separate agents, each with the same structure:

1. Query the RAG store with a relevant prompt (e.g. "interest rate for personal loan 700 credit score")
2. Receive top 3 policy chunks
3. Pass chunks + applicant data to GPT-4o-mini
4. Receive a complete email draft

**Approval agent** pulls: interest rates, loan terms, next steps, contact info.

**Rejection agent** pulls: rejection reason policy, alternative products, discount offers, legal disclaimers.

Human reviewer output also feeds into this layer — their decision triggers the same email composer, just with a human-verified decision as input.

### Layer 6 — RAG knowledge base
A local ChromaDB vector store built from your policy documents.

**What to put in your policy docs:**
- `interest_rates.txt` — rate tables by loan type, credit score band, loan amount
- `rejection_offers.txt` — alternative products, discount offers, what to give rejected applicants
- `legal_disclaimers.txt` — required legal language for approval and rejection emails
- `next_steps.txt` — what happens after approval, what docs are needed, timelines

Documents are chunked into ~500 token segments, embedded with OpenAI embeddings, and stored locally. The store only needs to be built once — subsequent runs load from disk.

### Layer 7 — Email composer
A single OpenAI GPT-4o-mini call with:
- System prompt defining the email format and tone
- Applicant data (name, loan amount, loan purpose, decision)
- RAG context (relevant policy chunks)
- Output: plain text email ready to send

### Layer 8 — Gmail sender
Uses the Gmail API (OAuth2) to send the composed email from your Google account. Fully automated after initial one-time browser authentication.

---

## Data flow summary

```
Application row
  → ML score + loan category
  → fraud_check() → CLEAN / REVIEW / AUTO_REJECT
  → [if CLEAN] retrieve RAG context
  → compose_email(applicant, decision, rag_context)
  → send_email(to, subject, body)
  → log result
```

---

## Key design decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Agent style | One-shot email | Simpler, safer, no state management needed |
| Vector store | ChromaDB (local) | No cloud dependency, free, Mac-compatible |
| LLM | GPT-4o-mini | Cheap (~$0.01/email), fast, good instruction following |
| Email provider | Gmail API | Free, easy OAuth, no extra service needed |
| Fraud detection | Rule-based | Transparent, auditable, easy to adjust thresholds |
| Human escalation | Hold + log | Agent never sends email for flagged cases |

---

## What the emails look like

### Approval email (RAG-grounded)
- Personalised greeting
- Approval confirmation with loan amount
- Interest rate pulled from RAG (based on credit score band + loan type)
- Loan term and monthly payment estimate
- Next steps (docs needed, timeline)
- Contact info

### Rejection email (RAG-grounded)
- Empathetic opening
- Clear rejection statement with general reason
- Alternative product offer pulled from RAG (e.g. secured loan, credit builder product)
- Discount or incentive if applicable
- Legal disclaimer pulled from RAG
- Encouragement to reapply

---

## Fraud escalation flow (human-in-the-loop)

```
Suspicious application detected
  → logged to console / file
  → pipeline holds (no email sent)
  → human reviews raw data + flag reason
  → human inputs: approve / reject / discard
  → decision feeds into email composer
  → email sent as normal
```

For v1, the "human input" can be as simple as editing a flag in a CSV or running a one-line script. A simple web UI or Slack notification can be added later.
