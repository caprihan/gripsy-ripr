# RIPR — Research, Iterate, Peer Review

> Multi-LLM fact-checking for AI-assisted content. Never publish something you'll regret.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Made with Claude](https://img.shields.io/badge/Made%20with-Claude-blueviolet)](https://claude.ai)

## The Problem

AI can write. AI can also hallucinate. When you publish content under your name, you need to know the facts are solid.

I learned this the hard way when Claude confidently fabricated statistics in a LinkedIn post. The post went viral... along with the corrections pointing out my "facts" were made up.

**RIPR is my solution:** Before any content goes public, three different AI models fact-check it independently. If they agree it's solid (≥9/10), publish. If not, iterate until they do.

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                      RIPR Pipeline                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  [1] RESEARCH                                                │
│      └─▶ Gather sources, extract claims, cite everything     │
│                                                              │
│  [2] VALIDATE (Multi-Model)                                  │
│      └─▶ Claude: Deep analysis with extended thinking        │
│      └─▶ ChatGPT: Independent verification via n8n           │
│      └─▶ Gemini: Third perspective via n8n                   │
│      └─▶ Aggregate scores using weighted rubric              │
│                                                              │
│  [3] ITERATE (if score < 9.0)                                │
│      └─▶ Compile feedback from all validators                │
│      └─▶ Revise based on specific issues raised              │
│      └─▶ Re-validate (max 3 iterations)                      │
│                                                              │
│  [4] OUTPUT                                                  │
│      └─▶ Verified content + confidence score                 │
│      └─▶ Audit trail (sources, scores, iterations)           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Why Three Models?

**Epistemic diversity.** Each LLM has different training data, different knowledge cutoffs, and different blind spots. When Claude, ChatGPT, and Gemini all agree a claim is verified, you can be confident. When they disagree, you've found something worth investigating.

## Scoring Rubric

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| **Accuracy** | 40% | Are claims factually correct and verifiable? |
| **Freshness** | 20% | Are sources current? Dates verified in article body? |
| **Completeness** | 20% | Do sources fully support the claims made? |
| **Relevance** | 10% | Do sources directly address the topic? |
| **Clarity** | 10% | Are claims unambiguous and well-stated? |

**Thresholds:**
- **≥9.0** — Approved for publication
- **7.0–8.9** — Needs revision (iterate with feedback)
- **<7.0** — Major issues (flag for human review)

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/code) or any Claude-based agent
- [n8n](https://n8n.io) (self-hosted or cloud) for external validators
- OpenAI API key (for ChatGPT validator)
- Google AI API key (for Gemini validator)

### 1. Clone the Skill

```bash
# Add to your Claude Code skills directory
cp -r gripsy-ripr ~/.claude/skills/
```

### 2. Import n8n Workflows

Import the workflow files from `n8n/` into your n8n instance:

1. `ripr-gpt-workflow.json` — ChatGPT validator
2. `ripr-gemini-workflow.json` — Gemini validator

Configure each workflow with your API keys, then **activate** them.

### 3. Update Config

Edit `config.json` with your webhook URLs:

```json
{
  "external_validators": {
    "chatgpt": {
      "enabled": true,
      "webhook": "https://your-n8n.com/webhook/ripr-gpt"
    },
    "gemini": {
      "enabled": true,
      "webhook": "https://your-n8n.com/webhook/ripr-gemini"
    }
  }
}
```

### 4. Test the Pipeline

```
You: RIPR this: "OpenAI was founded in 2015 by Sam Altman and Elon Musk."

Claude: Running RIPR validation...
- Claude: 9.2/10 ✓
- ChatGPT: 9.0/10 ✓  
- Gemini: 8.8/10 ✓
- Weighted: 9.0/10 — APPROVED

Verified claims:
✓ OpenAI founded in 2015 (Dec 11, 2015)
✓ Sam Altman was a co-founder
✓ Elon Musk was a co-founder (later departed board in 2018)
```

## RIPR Modes

### RIPR-Full (Default)
For content published under your name: LinkedIn posts, Medium articles, tweets.
- Full multi-model validation
- Iterative refinement until ≥9.0
- Human approval gate before publish

### RIPR-Lite
For lower-stakes content: daily digests, news summaries, internal reports.
- Single-pass verification
- Date and source checking only
- Auto-publish if checks pass

Configure in `config.json`:
```json
{
  "ripr_lite": {
    "enabled": true,
    "skip_external_validators": true,
    "auto_publish": true
  }
}
```

## Webhook API

The n8n validators expect this payload:

```json
POST /webhook/ripr-gpt
Content-Type: application/json

{
  "content": "The draft content to validate...",
  "claims": ["Claim 1", "Claim 2", "Claim 3"],
  "sources": ["https://source1.com", "https://source2.com"]
}
```

Response format:

```json
{
  "claim_validations": [
    {
      "claim_id": "C1",
      "status": "VERIFIED",
      "notes": "Explanation of verification..."
    }
  ],
  "scores": {
    "accuracy": 9,
    "freshness": 8,
    "completeness": 9,
    "relevance": 9,
    "clarity": 10
  },
  "weighted_score": 9.0,
  "feedback": [],
  "verdict": "PASS"
}
```

## Example Output

```json
{
  "id": "ripr-20260205-093000",
  "iterations": 2,
  "final_score": 9.2,
  "verdict": "APPROVED",
  "content": "The verified, publication-ready content...",
  "claims": [
    {
      "id": "C1",
      "text": "Anthropic raised $4B from Amazon",
      "source": "https://techcrunch.com/...",
      "status": "VERIFIED"
    }
  ],
  "scores": {
    "claude": {"accuracy": 9.5, "freshness": 9.0, ...},
    "chatgpt": {"accuracy": 9.0, "freshness": 9.5, ...},
    "gemini": {"accuracy": 9.2, "freshness": 8.8, ...}
  },
  "audit_trail": {
    "iteration_1": {"score": 8.4, "issues": ["Date unverified"]},
    "iteration_2": {"score": 9.2, "issues": []}
  }
}
```

## File Structure

```
gripsy-ripr/
├── SKILL.md              # Main orchestration instructions
├── README.md             # This file
├── config.json           # Configuration and thresholds
├── agents/
│   ├── researcher.md     # Research phase prompt
│   └── validator-claude.md # Claude validation prompt
├── rubric/
│   └── scoring.md        # Detailed scoring criteria
└── n8n/
    ├── ripr-gpt-workflow.json    # ChatGPT validator
    └── ripr-gemini-workflow.json # Gemini validator
```

## FAQ

**Q: Why not just use one model with a "be careful" prompt?**

A: Single-model fact-checking has a fundamental problem: the same blind spots that caused the hallucination will miss it during review. Multi-model validation catches errors that any single model would miss.

**Q: Is this overkill for a tweet?**

A: Maybe! Use RIPR-Lite for quick posts, RIPR-Full for anything that matters. The config is flexible.

**Q: What if the validators disagree?**

A: That's valuable signal. When models disagree, RIPR flags the claim for human review rather than auto-approving. Disagreement = uncertainty = investigate further.

**Q: Can I add more validators?**

A: Yes. Add new webhooks to `config.json` and update the scoring weights. The more diverse perspectives, the better.

## Credits

Built by [Gaurav Caprihan](https://linkedin.com/in/gauravcaprihan) with Claude.

Inspired by the "trust but verify" principle and too many embarrassing corrections.

## License

MIT — Use it, modify it, make it better.

---

**Get a grip on AI.** 🦎

[gripsy.com.au](https://gripsy.com.au)
