Title: Using Swiss hosted open weight LLMs with OpenCode
Tags: AI Agents, OpenCode, Infomaniak, Privacy
Language: en
Summary: Configuring OpenCode to use Infomaniak AI Tools' OpenAI-compatible API for sovereign, Swiss-hosted open weight LLMs.

When I started using LLM based AI agents I first tried [OpenCode] for the
following reasons:

 * It's open source, which I generally prefer
 * It works with plenty of different LLM providers

I initially used it with the default [OpenCode Zen] provider, but quickly hit
limits of the free plan. I then got myself a Claude Pro subscription and used
it with that. But at some point Anthropic disallowed using their subscriptions
with tools other than Claude Code.[^1]

After trying to use an Anthropic API key for some time, I quickly noticed how
much more expensive it is and switched to using Claude Code and kept using the
subscription.

[OpenCode Zen]: https://opencode.ai/zen
[^1]: See [discussion on HN](https://news.ycombinator.com/item?id=47444748)
    from that time, the [OpenCode Anthropic provider
    docs](https://opencode.ai/docs/providers#anthropic)
    > There are plugins that allow you to use your Claude Pro/Max models with
    > OpenCode. Anthropic explicitly prohibits this.

    and the [Anthropic ToS](https://www.anthropic.com/legal/consumer-terms)
    > You may not access or use, or help another person to access or use, our
    > Services in the following ways:
    > ...
    > 7. Except when you are accessing our Services via an Anthropic API Key or
    >    where we otherwise explicitly permit it, to access the Services
    >    through automated or non-human means, whether through a bot, script,
    >    or otherwise.

But this whole thing made me realize how dependent I am on a single company
which almost forces me to use their proprietary tool and I'm also sending quite
a lot of data to them through my use of their services.

All that led me to research which providers of LLMs exist in Switzerland.
<https://mydata.ch/schweizer-ki-anbieter> gives a nice overview of some
existing providers. Infomaniak stood out because they have an affordable pay by
use plan and have some decent open weight models available.

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

Add the following to your `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "infomaniak": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Infomaniak AI Tools",
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

Run `/connect` in the OpenCode TUI


<figure>
<img src="{static}/images/opencode_infomaniak/opencode_connect_provider.png" alt="OpenCode TUI /connect autocompletion" width="100%">
</figure>

and select our custom provider.

<figure>
<img src="{static}/images/opencode_infomaniak/opencode_choose_provider.png" alt="OpenCode TUI select provider" width="100%">
</figure>

In the next screen paste your API token which you can create with the
[Infomaniak Manager] as well.

# Using It

My first impression was that it feels faster than using Claude Code with the
Opus models. According to benchmarks Kimi 2.6 should be comparable to the Opus
4.x models.[^2]

I had one occasion where it seemed to block for around a minute on a very
simple task. Maybe the server was overloaded there? Other than that it felt
faster than using Opus 4.8 with my Claude Pro subscription.

<figure>
<img src="{static}/images/opencode_infomaniak/opencode_running.png" alt="OpenCode running with Kimi-K2.6" width="100%">
<figcaption>OpenCode running with Kimi K2.6 supporting me with this blog post</figcaption>
</figure>

What I'm missing from Claude are:

 * That it uses interactive forms heavily during plan mode to ask me questions
   where I can just select one of the suggested answers or provide my own.
 * [Auto-mode], permission fatigue is real and together with nono.sh sandboxing
   auto mode feels like a very good compromise to me.[^3]

<figure>
<img src="{static}/images/opencode_infomaniak/claude_code_questions.png" alt="Claude Code asking questions" width="100%">
<figcaption>Claude Code asking questions before going ahead.</figcaption>
</figure>

I guess there are plugins for OpenCode to achieve a similar feature set as I'm
used to from Claude.
<https://github.com/the-perfect-developer/the-perfect-opencode/tree/main> looks
promising for example.

Also I'm not sure what differences in the experience are due to the different
LLM and which due to the changed harness. To more objectively compare the
models I should probably use both with OpenCode.

[Auto-Mode]: https://claude.com/blog/auto-mode
[^2]: See <https://benchlm.ai/compare/claude-opus-4-6-vs-kimi-2-6>, <https://codersera.com/blog/kimi-k2-6-vs-claude-opus-4-7/>, <https://codingfleet.com/blog/claude-opus-4-8-vs-kimi-k2-6/>
[^3]: See my previous blog post [Safe Ways to Use AI Agents](https://blog.rnstlr.ch/safe-ways-to-use-ai-agents.html) on the topic of sandboxing

# Summary

I love being able to use a Swiss hosted open weight LLM for the following reasons:

 * **Digital sovereignty**: I feel less dependent on a single American company
 * **No vendor lock-in**: I can freely choose which agentic CLI to use
 * **Supporting a domestic company**: I like to support local businesses

While I miss some of the power Claude Code gives me with their frontier models,
I think in the long-term I'll benefit more from using open weight models with
open tools.
