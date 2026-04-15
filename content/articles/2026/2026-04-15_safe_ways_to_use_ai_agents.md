Title: Safe Ways to Use AI agents
Tags: AI Agents, Sandboxing
Language: en
Status: draft
Summary: Investigating ways to easily isolate AI agents to reduce risks in running them.

At [Renuo] we started using AI coding agents (like [Claude Code], [OpenCode] or
[Antigravity]) for development, and I also started using them for personal
projects like my [Raspberry Dashboard]. Additionally we started building our own
AI agent which is integrated in [Redmine], our ticketing and project
management system.

During this, we became aware of the security risks involved: by default these
agents run with full user permissions -- they can read and write files, execute
commands, and access credentials on the host system.

Johann Rehberger's 39c3 talk [Agentic ProbLLMs: Exploiting AI Computer-Use
and Coding Agents][39c3-talk] shows how prompt injection can lead to remote
code execution and credential exfiltration in agents like Claude Code and
GitHub Copilot. I recommend watching it.

In this post we'll take a look at these risks and the pragmatic solutions we
came up with to keep a balance between developer experience and security.

[Renuo]: https://www.renuo.ch
[Claude Code]: https://code.claude.com/docs/
[OpenCode]: https://opencode.ai/
[Antigravity]: https://antigravity.google/
[Raspberry Dashboard]: https://github.com/rnestler/raspberry-dashboard
[Redmine]: https://www.redmine.org/
[39c3-talk]: https://media.ccc.de/v/39c3-agentic-probllms-exploiting-ai-computer-use-and-coding-agents

# Risks of AI Agents

> LLMs are probabilistic -- a 1% chance of disaster makes it a matter of when,
> not if.
> -- [Agent Safehouse](https://agent-safehouse.dev/)

Most AI coding agents run with the same permissions as the user who started
them. They have access to the file system, can execute arbitrary shell commands,
and inherit all credentials available in the environment. Since LLMs are
susceptible to prompt injection -- malicious instructions hidden in code,
documentation, or web content -- this creates a real attack surface.

<figure>
<img src="{static}/images/ai_agents/agentic-probllms-screenshot-promt-injection.png" alt="Prompt Injection TTPs overview from Johann Rehberger's talk" width="100%">
<figcaption>Image source: <a href="https://embracethered.com/">embracethered.com</a> by Johann Rehberger</figcaption>
</figure>

The risks boil down to a few categories:

 *  **Exposing credentials**: Agents have access to environment variables,
    config files, and credential stores. A prompt injection can trick an agent
    into exfiltrating API keys or access tokens to an attacker-controlled
    server.

 * **Malware installation**: Agents can be tricked into downloading and
   executing malicious code, for example through poisoned dependencies or
   malicious instructions in README files.

 * **Destructive actions on the local machine**: An agent might delete files,
   overwrite configurations, or corrupt a local database -- through prompt
   injection or simply by making a wrong decision.

 * **Destructive actions on remote systems**: Agents often have access to CLI
   tools that interact with production infrastructure.

I asked some developer what incidents they had happen with AI agents so far.
Here are a few examples:

> I don't have the session anymore, but while working on a redmine integration,
> it found out that I had a REDMINE_API_KEY in my ENV variables and started
> fetching data from our production redmine.
> -- Alesandro Rodi

> While it wasn't a major issue, it was frustrating when database migration
> errors caused the development database to be deleted and recreated, as I
> often lost data I needed to keep.
> -- Bruno Costanzo

> While I was testing our claude code skill to deploy web-apps to deplo.io, the
> agent hit the quota limit of the number of apps. To solve this it decided
> it's best to delete existing apps with `nctl delete app`. Luckily it asked
> for confirmation before going ahead.
> -- Josua Schmid

We didn't have a case yet were things went seriously wrong, mostly because we
don't let the agents run unattended. But it was enough to trigger us to really
think about how to improve the situation.

For a deeper look at these attack vectors see the [39c3 talk][39c3-talk]
referenced in the introduction.

# Mitigation Strategies

 * Trade off between developer convenience and security / safety
    * Permission fatigue
    * Local setup vs. needing to maintain a functional sandbox environement / Dockerfile
 * Manual approval: Permission fatigue, configuration errors
 * Agent specific configuration: Bash commands can find tricky ways to
   circumvent stuff. No protection against web injected remote code execution
 * Ways of isolation
    * Docker containers / Devcontainers. Mention gfrörli API PR
      https://github.com/gfroerli/api/pull/356
    * Sandboxing
        * builtin (claude)
        * agent safehouse
        * nono.sh

## Pragmatic Sandboxing

 * For custom agents
    * Use Docker containers and limit availability of secrets
    * Set things up outside of agent (example clone the repo, install
      dependencies)
 * For dev machines
    * Use sandboxing tools with out-of-the-box profiles for common AI agents

# Further Resources

 * https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html

# Notes

## Resources

 * https://docs.google.com/document/d/1ONlU5b8wehuoehDSsaHrosZ0FrByEJdl9DuUMW8qGiI/edit?tab=t.0
 * https://media.ccc.de/v/39c3-agentic-probllms-exploiting-ai-computer-use-and-coding-agents

## Topics

 * What can go wrong
