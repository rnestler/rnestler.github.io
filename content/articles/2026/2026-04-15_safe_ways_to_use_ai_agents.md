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

 * We started using AI coding agents and also creating agents to automate tasks
 * During this we got aware of risks, I can also recommend watching the
   "Agentic ProbLLMs: Exploiting AI Computer-Use and Coding Agents" by Johann
   Rehberger at the 39c3 conference
 * This is a pragmatic solution that helps developers isolate agents while
   still keeping things simple.

## Risks of AI Agents

 * Summarize findings of "Agentic ProbLLMs: Exploiting AI Computer-Use and Coding Agents"
 * Simplify them into
   * Exposing credentials
   * Malware installs
   * Destructive actions on local machine
   * Destructive actions on remove machines (nctl delete apps example)
