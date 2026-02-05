# RIPR Setup Guide

Step-by-step instructions to get RIPR running.

## Prerequisites

- **Claude Code** or compatible Claude-based agent (e.g., OpenClaw)
- **n8n** instance (self-hosted or cloud)
- **OpenAI API key** (for ChatGPT validator)
- **Google AI API key** (for Gemini validator)

## Step 1: Copy the Skill

Copy the `gripsy-ripr` folder to your agent's skills directory:

```bash
# For Claude Code
cp -r gripsy-ripr ~/.claude/skills/

# For OpenClaw
cp -r gripsy-ripr ~/clawd/skills/
```

## Step 2: Set Up n8n Workflows

### 2.1 Import ChatGPT Validator

1. Open your n8n instance
2. Go to **Workflows** → **Import from File**
3. Select `n8n/ripr-gpt-workflow.json`
4. Configure credentials:
   - Create an **OpenAI API** credential with your API key
   - Link it to the "ChatGPT Validator" node
5. **Activate** the workflow (toggle in top-right)
6. Note your webhook URL: `https://your-n8n.com/webhook/ripr-gpt`

### 2.2 Import Gemini Validator

1. Import `n8n/ripr-gemini-workflow.json`
2. Configure credentials:
   - Create a **Google Palm/Gemini API** credential with your API key
   - Link it to the "Gemini Validator" node
3. **Activate** the workflow
4. Note your webhook URL: `https://your-n8n.com/webhook/ripr-gemini`

## Step 3: Configure RIPR

1. Copy the template config:
   ```bash
   cp config.template.json config.json
   ```

2. Edit `config.json` with your webhook URLs:
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

## Step 4: Test the Setup

### Test ChatGPT Validator

```bash
curl -X POST "https://your-n8n.com/webhook/ripr-gpt" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Test claim: The sky is blue.",
    "claims": ["The sky is blue"],
    "sources": ["common knowledge"]
  }'
```

Expected response:
```json
{
  "validator": "chatgpt",
  "claim_validations": [...],
  "scores": {...},
  "weighted_score": 9.5,
  "verdict": "PASS"
}
```

### Test Gemini Validator

```bash
curl -X POST "https://your-n8n.com/webhook/ripr-gemini" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Test claim: The sky is blue.",
    "claims": ["The sky is blue"],
    "sources": ["common knowledge"]
  }'
```

## Step 5: Run RIPR

In your Claude session:

```
You: RIPR this: "Anthropic raised $4 billion from Amazon in 2024."

Claude: Running RIPR validation...

Research complete. Found 3 sources.
- Claude validation: 9.2/10
- ChatGPT validation: 9.0/10
- Gemini validation: 9.1/10

Weighted score: 9.1/10 — APPROVED ✓

Verified claims:
✓ Amazon invested $4B in Anthropic (Sep 2024)
✓ This was Amazon's largest venture investment

Sources:
1. https://techcrunch.com/2024/09/...
2. https://www.reuters.com/2024/09/...
```

## Troubleshooting

### Webhook returns 404

The workflow isn't activated. In n8n:
1. Open the workflow
2. Click the toggle in the top-right to activate
3. Green = active, gray = inactive

### "Invalid API key" errors

Check your credentials in n8n:
1. Go to **Credentials**
2. Find the OpenAI/Gemini credential
3. Verify the API key is correct

### Gemini returns malformed JSON

The workflow includes a parser that handles markdown-wrapped responses. If it still fails, check:
1. Your Gemini API quota
2. That `responseMimeType: "application/json"` is set

### Low scores on valid claims

The validators are conservative by default. If a claim is correct but getting low scores:
1. Add more specific sources
2. Ensure sources directly mention the claim
3. Check if the claim needs date context

## Optional: Disable a Validator

If you only want two validators:

```json
{
  "external_validators": {
    "chatgpt": {
      "enabled": true,
      ...
    },
    "gemini": {
      "enabled": false,
      ...
    }
  }
}
```

RIPR will use only the enabled validators.

## Need Help?

- GitHub Issues: [github.com/caprihan/gripsy-ripr](https://github.com/caprihan/gripsy-ripr)
- Twitter: [@GCaprihan](https://x.com/GCaprihan)
