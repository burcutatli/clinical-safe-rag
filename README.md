# Clinical-Safe RAG

A retrieval-augmented generation (RAG) chatbot with **architectural clinical-safety routing**, built for a telehealth customer support scenario.

The core idea: medical questions should never be answered by a general-purpose LLM, no matter how good the prompt is. This demo routes clinical queries to direct human escalation **before** they reach the main model — a structural guardrail, not a prompt-level one.

## Why this architecture

Most chatbot demos rely on a single prompt with instructions like *"if the question is clinical, refer to a doctor."* That works 90% of the time. In a regulated domain like telehealth, 90% is a compliance incident waiting to happen.

This system takes a different approach:

```
User question
     ↓
Haiku classifier  (fast, cheap: ~$0.0003/call)
     ↓
├── Clinical?  →  Direct escalation template. No RAG. No Sonnet.
└── Everything else  →  Chroma retrieval → Sonnet with context
```

The classifier intercepts medical queries before the expensive, generative model is ever invoked. Clinical safety becomes a **property of the system**, not a hope about model behaviour.

## What's in the demo

- **5 synthetic support documents** (shipping, refunds, semaglutide, Texas regulations, billing)
- **Chroma vector database** with automatic embedding (all-MiniLM-L6-v2)
- **Haiku-based query classifier** (5 categories: shipping, billing, account, clinical, general)
- **Sonnet-based answer generation** with a strict *context-only* system prompt
- **Three test cases** demonstrating:
  1. A normal support question → retrieved + answered correctly
  2. An unsupported question (loyalty program) → graceful "I don't know"
  3. A clinical question (side effects, dosage change) → routed to escalation, LLM never touched

## Cost characteristics

| Query type | Components called | Approx. cost |
|---|---|---|
| Clinical | Haiku classifier only | **$0.0003** |
| Non-clinical | Haiku + Chroma + Sonnet | ~$0.011 |

Clinical routing isn't just a safety feature — it's also a **cost lever**. Roughly 5-7% of telehealth support volume is clinical; routing those queries bypasses the expensive Sonnet call entirely.

## Stack

- **Claude API** (Sonnet 4.5 for generation, Haiku 4.5 for classification)
- **ChromaDB** (in-memory vector database)
- **Python** (no framework — plain API calls for clarity)

## Run it yourself

Open `clinical_safe_rag_demo.ipynb` in Google Colab ([one-click link in the notebook](clinical_safe_rag_demo.ipynb)).

You'll need:
1. An Anthropic API key ([console.anthropic.com](https://console.anthropic.com))
2. Add it to Colab Secrets as `ANTHROPIC_API_KEY`

Then run all cells top-to-bottom. No setup beyond `pip install`.

## What this is not

- Not a production system — no conversation state, no PII masking, no HIPAA-compliant logging
- Not a full implementation — real clinical routing needs human-in-the-loop review, audit trails, and state-specific filtering
- Not a LangChain wrapper — intentionally built on raw API calls to show what's actually happening

## What this is

A working proof-of-concept that demonstrates **architectural guardrails over prompt-level hope**. The kind of pattern you'd propose to a compliance-sensitive client in a first architecture discussion.

---

Built by [Burcu Tatlı](https://www.linkedin.com/in/burcutatli) · AI Operations & Automation  
Portfolio: [portfolioburcu.netlify.app](https://portfolioburcu.netlify.app)

