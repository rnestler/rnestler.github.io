Title: Using Swiss hosted open weight LLMs with OpenCode
Tags: AI Agents, OpenCode, Infomaniak, Privacy
Language: en
Summary: Configuring OpenCode to use Infomaniak AI Tools' OpenAI-compatible API for sovereign, Swiss-hosted open weight LLMs.
Status: draft

[Motivation]

[Why Infomaniak AI Tools]

# Prerequisites

You will need the following:

 * An [Infomaniak] account with AI Services enabled (1 million free credits are included)
 * Your AI `product_id` from the [Infomaniak Manager]
 * An API token scoped to AI Services
 * [OpenCode] installed

[Infomaniak]: https://www.infomaniak.com/en/hosting/ai-services
[Infomaniak Manager]: https://manager.infomaniak.com
[OpenCode]: https://opencode.ai/

# Configuring the Provider

Run `/connect` in the OpenCode TUI, scroll down to **Other**, choose a provider ID like `infomaniak`, and paste your API token.

Then add the following to your `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "infomaniak": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Infomaniak AI",
      "options": {
        "baseURL": "https://api.infomaniak.com/2/ai/{product_id}/openai/v1"
      },
      "models": {
        "moonshotai/Kimi-K2.6": {
          "name": "Kimi K2.6",
          "limit": { "context": 256000, "output": 32768 }
        }
      }
    }
  }
}
```

Make sure to replace `{product_id}` with the numeric ID of your AI Services product.

[Gotchas / Notes]

# Using It

[Personal experience with Kimi K2.6]

# Summary

[Recap and links]
