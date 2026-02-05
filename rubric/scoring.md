# RIPR Scoring Rubric

## Overview

RIPR uses a weighted multi-dimensional scoring system validated by multiple AI models. The final score determines whether content is ready for publication.

## Dimensions & Weights

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| Accuracy | 40% | Factual correctness, source quality |
| Freshness | 20% | Currency of information, date verification |
| Completeness | 20% | Coverage depth, context inclusion |
| Relevance | 10% | Audience fit, platform appropriateness |
| Clarity | 10% | Structure, readability, flow |

## Scoring Scale

Each dimension scored 1.0 to 10.0:

| Score | Meaning |
|-------|---------|
| 10.0 | Exceptional — publication ready, exemplary |
| 9.0-9.9 | Excellent — meets high professional standards |
| 8.0-8.9 | Good — minor improvements possible |
| 7.0-7.9 | Adequate — needs some revision |
| 6.0-6.9 | Below standard — significant revision needed |
| <6.0 | Unacceptable — major issues or rewrite needed |

## Publication Threshold

**Minimum score for publication: 9.0**

This threshold ensures:
- All claims are verified or confirmed
- Information is current and relevant
- No critical issues remain
- Content reflects professional quality

## Score Calculation

### Single Validator Score
```
weighted_score = 
  (accuracy × 0.40) +
  (freshness × 0.20) +
  (completeness × 0.20) +
  (relevance × 0.10) +
  (clarity × 0.10)
```

### Multi-Validator Aggregation
```
final_score = (claude_score + gpt_score + gemini_score) / 3
```

All validators weighted equally to ensure epistemic diversity.

## Dimension Details

### Accuracy (40%)

**What it measures:**
- Are all factual claims correct?
- Are sources authoritative and reliable?
- Are quotes accurately attributed?
- Are statistics properly contextualized?

**Scoring guide:**

| Score | Criteria |
|-------|----------|
| 10 | All claims verified with 2+ authoritative sources |
| 9 | All claims verified, sources are solid tier-1/primary |
| 8 | Claims verified, 1-2 rely on single credible source |
| 7 | Most claims verified, some sourcing could be stronger |
| 6 | Multiple claims inadequately sourced |
| 5 | Accuracy concerns, unverified claims present |
| <5 | Factual errors or contradicted claims present |

**Red flags (automatic score cap at 5):**
- Any claim contradicted by authoritative source
- Fabricated quotes or statistics
- Misattributed statements
- Satire/joke source used as fact

### Freshness (20%)

**What it measures:**
- Is the information current?
- Are publication dates verified?
- Is context time-appropriate?

**Scoring guide:**

| Score | Criteria |
|-------|----------|
| 10 | All sources current, all dates explicitly verified |
| 9 | Current sources, dates verified, minor lag acceptable |
| 8 | Mostly current, one older source used appropriately |
| 7 | Some sources older but context explains |
| 6 | Freshness concerns for time-sensitive content |
| 5 | Old information presented without time context |
| <5 | Stale information presented as current |

**Red flags (automatic score cap at 5):**
- Date on source cannot be verified
- Breaking news context uses week-old sources
- "Latest" or "recent" claims with old data

### Completeness (20%)

**What it measures:**
- Is the topic adequately covered?
- Is necessary context included?
- Are multiple perspectives represented (if relevant)?

**Scoring guide:**

| Score | Criteria |
|-------|----------|
| 10 | Comprehensive coverage, all key angles addressed |
| 9 | Thorough, only minor additional context possible |
| 8 | Good coverage, one aspect could be deeper |
| 7 | Adequate, missing some useful context |
| 6 | Notable gaps in coverage |
| 5 | Important aspects missing |
| <5 | Major omissions that mislead reader |

**Red flags (automatic score cap at 5):**
- One-sided coverage of controversial topic
- Missing crucial context that changes meaning
- Key stakeholder perspective absent

### Relevance (10%)

**What it measures:**
- Does content match target audience?
- Is it appropriate for the platform?
- Does it align with stated purpose?

**Scoring guide:**

| Score | Criteria |
|-------|----------|
| 10 | Perfectly targeted, ideal for platform and audience |
| 9 | Well-targeted, minor optimization possible |
| 8 | Relevant, slight tangents don't detract |
| 7 | Mostly relevant, some off-topic content |
| 6 | Partially misaligned with audience |
| 5 | Significant relevance issues |
| <5 | Wrong audience or platform fit |

### Clarity (10%)

**What it measures:**
- Is the structure logical?
- Is the writing clear and concise?
- Does it flow well?

**Scoring guide:**

| Score | Criteria |
|-------|----------|
| 10 | Crystal clear, perfect structure, engaging |
| 9 | Clear and well-structured, minor polish possible |
| 8 | Clear, some sentences could be tighter |
| 7 | Understandable but verbose or awkward spots |
| 6 | Confusing in parts, structure issues |
| 5 | Unclear writing, poor organization |
| <5 | Incomprehensible or badly structured |

## Verdict Logic

Based on final aggregated score:

| Final Score | Verdict | Action |
|-------------|---------|--------|
| ≥ 9.0 | PASS | Ready for publication |
| 8.0 - 8.9 | REVISE | Minor fixes needed, re-validate |
| 7.0 - 7.9 | REVISE | Moderate revision needed |
| < 7.0 | FAIL | Major rewrite or abandon |

**Override conditions:**
- Any CONTRADICTED claim → FAIL regardless of score
- Any critical feedback → REVISE regardless of score
- 3 iterations reached → Present best version with warning

## Iteration Limits

- **Maximum iterations**: 3
- After 3 iterations, present best achieved version
- Include warning if still below 9.0
- Require explicit human approval for sub-threshold publication
