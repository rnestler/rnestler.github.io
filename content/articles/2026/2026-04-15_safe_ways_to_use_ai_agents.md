Title: Safe Ways to Use AI agents
Tags: AI Agents, Sandboxing
Language: en
Status: draft
Summary: Investigating ways to easily isolate AI agents to reduce risks in running them.

# Notes

## Resources

 * https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html
 * https://docs.google.com/document/d/1ONlU5b8wehuoehDSsaHrosZ0FrByEJdl9DuUMW8qGiI/edit?tab=t.0
 * https://media.ccc.de/v/39c3-agentic-probllms-exploiting-ai-computer-use-and-coding-agents

## Topics

 * By default AI agents have *full* system access
 * AI agents are easily tricked into downloading and running malware
 * What can go wrong
 * Trade off between developer convenience and security / safety
   * Permission fatigue
   * Local setup vs. needing to maintain a functional sandbox environement / Dockerfile
 * Ways of isolation
   * Docker containers / Devcontainers. Mention gfrörli API PR https://github.com/gfroerli/api/pull/356
   * Sandboxing (builtin (claude) agent safehouse, nono.sh)


## Introduction

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



## Risks of AI Agents

 * Summarize findings of "Agentic ProbLLMs: Exploiting AI Computer-Use and Coding Agents"
 * Simplify them into
   * Exposing credentials
   * Malware installs
   * Destructive actions on local machine
   * Destructive actions on remove machines (nctl delete apps example)
