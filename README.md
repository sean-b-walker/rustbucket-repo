# rustbucket

*It's not magic, it's math. Agentic workflows vs vibes*

A harness-agnostic collection of AI agents, plus the contracts,
integrations, and docs that make them work. The repo is a library:
pick the pieces you want and wire them into whichever harness you use.

## The idea

Agents shouldn't be tied to one tool. Each piece here is written so it
can run in any harness that supports it:

- **Agents** say *what* to do, and name the capabilities they need.
- **Contracts** define what each capability must do, independent of who
  provides it.
- **Integrations** are one way of providing a capability, such as an
  MCP server for a specific service.

Swap an integration without touching the agent. Move to a new harness
without rewriting the agent.

## What's here

    agents/         Agent definitions (AGENT.md per agent)
    contracts/      Capability and data-format specs
    integrations/   Implementations of those capabilities
    docs/           Notes and guides (coming)
    scripts/        Install and helper scripts (coming)
