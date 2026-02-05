# RIPR — Research, Iterate, Peer Review

**By Gripsy** | Version 2.2.0

> "Never publish something you'll regret. Multi-LLM fact-checking for content."

## Overview

RIPR is a **verification pipeline**. It takes content with factual claims and validates them through multi-LLM peer review. It is agnostic of what you do with the content afterward — posting to LinkedIn, publishing on Medium, or just keeping it in a report.

**RIPR's job:** Ensure claims are accurate, sourced, and verified.
**Not RIPR's job:** Formatting, platform rules, posting.

---

## Personas

RIPR adapts its voice, depth, and focus based on the selected persona.

### Default: Technical Writer / Content Creator
- **Use when:** LinkedIn posts, X threads, Medium articles, tech commentary
- **Voice:** Clear, accessible, engaging
- **Focus:** Accuracy of technical claims, proper attribution, balanced perspective
- **Depth:** Practitioner-level (explains concepts, doesn't assume deep domain expertise)

### Business Consultant (McKinsey/BCG Partner)
- **Trigger:** "RIPR as consultant", "RIPR business mode", "executive lens"
- **Voice:** Strategic, authoritative, insight-driven
- **Focus:** Market implications, competitive dynamics, executive-level insights
- **Depth:** C-suite ready (assumes business acumen, connects to strategy/value)
- **Qualities:**
  - Frames insights around business impact and ROI
  - Uses structured thinking (e.g., "Three forces at play...")
  - Cites market data, analyst reports, earnings calls
  - Addresses "so what?" for decision-makers

### Industry Verticals (Optional Modifier)
Add vertical expertise to any persona for domain-specific validation:

| Vertical | Focus Areas | Key Sources |
|----------|-------------|-------------|
| **Semiconductors** | Foundry dynamics, chip architecture, supply chain, geopolitics | SemiAnalysis, AnandTech, earnings calls |
| **Cloud Services** | Hyperscaler strategies, pricing, migration, multi-cloud | Gartner, Forrester, cloud vendor blogs |
| **Mining & Resources** | Commodity cycles, ESG, automation, critical minerals | S&P Global, company filings, trade journals |
| **Financial Services** | Regulation, fintech disruption, risk, digital transformation | FT, Bloomberg, central bank publications |
| **Healthcare/Pharma** | Clinical trials, FDA/TGA, digital health, AI in diagnostics | NEJM, Lancet, FDA announcements |

**Usage:** "RIPR as consultant, semiconductor vertical" or "RIPR this for mining executives"

---

## Context Gathering (Interactive Mode)

When RIPR is invoked without sufficient context, **ask clarifying questions ONE AT A TIME** — conversational style, not a batch of questions.

### Question Sequence

Ask these in order, waiting for each response before asking the next:

1. **Platform** → "What platform — LinkedIn, X, Medium, or internal doc?"
2. **Persona** → "Tech content creator (default) or business consultant lens?"
3. **Vertical** *(only if consultant)* → "Any industry focus? (semiconductors, cloud, mining, fintech...)"
4. **Audience** *(if still unclear)* → "Who's the target reader?"

### Example Flow (One Question at a Time)

```
User: "RIPR this piece on AI in mining"

Agent: "What platform — LinkedIn, X, Medium, or internal doc?"

User: "LinkedIn"

Agent: "Tech content creator lens, or business consultant (executive strategy)?"

User: "Consultant"

Agent: "Any specific industry vertical — mining, cloud, semiconductors?"

User: "Mining"

Agent: "Perfect. Mining exec audience, consultant lens, LinkedIn format. Starting RIPR..."
```

### Skip Questions When
- User explicitly provides context upfront ("RIPR this LinkedIn post as a tech writer for mining execs")
- Content is clearly simple/low-stakes
- User says "just RIPR it" or "quick check" (use defaults)

### One-Shot Override
If user provides all context in their initial request, confirm and proceed — no redundant questions.

---

## When RIPR Activates

**RIPR-Full** (multi-agent, iterative):
- Any content with factual claims that will be published under the user's name
- Triggered by: "RIPR this:", "fact-check this", "verify before posting"

**RIPR-Lite** (single-pass verification):
- Daily bulletins and digests
- Curated news summaries
- Lower-stakes content
- Triggered by: bulletin cron jobs, news aggregation tasks

## RIPR-Full Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                      RIPR-Full Flow                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  [1] RESEARCH                                                │
│      └─▶ Gather sources via web_search + web_fetch           │
│      └─▶ Extract factual claims from content                 │
│      └─▶ Link each claim to sources                          │
│      └─▶ Output: { claims[], sources[], content }            │
│                                                              │
│  [2] VALIDATE (Multi-Model)                                  │
│      └─▶ Claude: Score each claim                            │
│      └─▶ ChatGPT (n8n): Independent verification             │
│      └─▶ Gemini (n8n): Third perspective                     │
│      └─▶ Aggregate scores using rubric weights               │
│                                                              │
│  [3] ITERATE (if score < 9.0)                                │
│      └─▶ Compile feedback from all validators                │
│      └─▶ Revise content/claims based on feedback             │
│      └─▶ Re-validate (max 3 iterations)                      │
│                                                              │
│  [4] OUTPUT                                                  │
│      └─▶ Verified content                                    │
│      └─▶ Claims with verification status                     │
│      └─▶ Confidence score (0-10)                             │
│      └─▶ Audit trail (sources, scores, iterations)           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## RIPR-Lite Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                      RIPR-Lite Flow                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  [1] VERIFY DATES                                            │
│      └─▶ Fetch each source article                           │
│      └─▶ Confirm publication date in article body            │
│      └─▶ Reject if date cannot be verified                   │
│                                                              │
│  [2] CHECK SOURCES                                           │
│      └─▶ Verify source is reputable (not satire/spam)        │
│      └─▶ Confirm URL is accessible                           │
│      └─▶ Cross-reference key claims if extraordinary         │
│                                                              │
│  [3] OUTPUT                                                  │
│      └─▶ Verified content with source links                  │
│      └─▶ Confidence indicator                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Validation Endpoints

**Claude:** Native (built-in, use thinking=high for complex claims)

**ChatGPT:**
```
POST https://n8n.gauravcaprihan.com/webhook/ripr-gpt
Content-Type: application/json

{
  "content": "The draft content...",
  "claims": ["Claim 1", "Claim 2"],
  "sources": ["https://source1.com", "https://source2.com"]
}
```

**Gemini:**
```
POST https://n8n.gauravcaprihan.com/webhook/ripr-gemini
Content-Type: application/json

{
  "content": "The draft content...",
  "claims": ["Claim 1", "Claim 2"],
  "sources": ["https://source1.com", "https://source2.com"]
}
```

## Scoring Rubric

| Dimension | Weight | Description |
|-----------|--------|-------------|
| Accuracy | 40% | Are claims factually correct? |
| Freshness | 20% | Are sources current and dates verified? |
| Completeness | 20% | Are claims fully supported by sources? |
| Relevance | 10% | Do sources directly support the claims? |
| Clarity | 10% | Are claims unambiguous? |

**Thresholds:**
- **≥9.0**: Approved for publication
- **7.0-8.9**: Revise (iterate with feedback)
- **<7.0**: Major revision needed (flag for human review)

## Output Format

### RIPR Report Structure
```json
{
  "id": "ripr-20260205-011800",
  "iterations": 2,
  "final_score": 9.2,
  "verdict": "APPROVED",
  "content": "The verified content...",
  "claims": [
    {
      "id": "C1",
      "text": "Claim text here",
      "source": "https://...",
      "status": "VERIFIED",
      "notes": "Validator notes..."
    }
  ],
  "scores": {
    "claude": { "accuracy": 9.5, "freshness": 9.0, "completeness": 9.2, "relevance": 9.0, "clarity": 9.5 },
    "chatgpt": { "accuracy": 9.0, "freshness": 9.5, "completeness": 8.8, "relevance": 9.2, "clarity": 9.0 },
    "gemini": { "accuracy": 9.2, "freshness": 8.8, "completeness": 9.0, "relevance": 9.0, "clarity": 9.3 }
  },
  "feedback": [
    { "validator": "gemini", "type": "suggestion", "text": "Consider adding date context..." }
  ],
  "sources": [
    { "url": "https://...", "title": "Article Title", "date": "2026-02-04", "verified": true }
  ]
}
```

## Error Handling

- **Source unavailable**: Note in audit, reduce completeness score
- **Validator timeout**: Continue with available validators, note in audit
- **Max iterations reached**: Present best version with warning, require manual review
- **Validators disagree**: Flag for human judgment, include all perspectives

## Related Files

- `agents/researcher.md` — Research agent prompt
- `agents/validator-claude.md` — Claude validator prompt
- `rubric/scoring.md` — Scoring criteria details
- `config.json` — Runtime configuration
- `n8n/` — Workflow export files for validators
