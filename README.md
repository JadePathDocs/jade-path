# Jade Path

Jade Path is a proof-of-concept system for neurodiversity-aware AI assistants, 
built around persistent memory, low-pressure task support, and gentle 
re-entry after periods of low activity. It grew out of practical lessons 
in productivity support, context management, and everyday use.

The core of the system is a primary agent (Cara), shared 
tone and profile conventions, and supporting documentation covering the 
design decisions behind them. A Japanese language study agent (Erika) is also provided, which serves as an example of modular agent design that can work with the system.

## Repo Structure

- `README.md` — this file
- `agents/` — genericized agent instructions and supporting docs
  - `tone-calibration.md` — shared tone/voice conventions across agents
  - `user-profile.md` — generic user profile template
  - `Cara/` — project instructions and session-log format
  - `Erika/` — project instructions and progress-log format
- `docs/agent-design.md` — what the agents are, how scoped identity 
  documents work, core design principles
- `docs/context-management.md` — context budget decisions, document 
  trimming rationale, platform behavior observed in testing
- `docs/memory-architecture.md` — memory format evolution and tradeoffs 
  (stub, planned for development)
- `docs/infrastructure.md` — self-hosted stack and provider setup 
  (stub, planned for development)

## Quick Start

1. Copy the relevant agent folder from `agents/` (start with `Cara/` or 
   `Erika/` depending on your use case).
2. Paste the project instructions file into your platform of choice as a 
   system prompt or space/project instruction set.
3. Fill in `user-profile.md` with your own context, using the template 
   fields as a guide. Keep entries factual and scoped — see Privacy & Data Handling below.
4. If using a persistent memory store (e.g. Notion via MCP), connect it 
   and point the agent at your profile and log files.

This is a manual setup process at this project's early stage. A formal user onboarding feature is planned for future development.

## Design Philosophy

This project is built around users who benefit from externalized 
structure, persistent context, and low-pressure task support rather than 
motivation- or pressure-based productivity systems. The agents are 
designed to reduce cognitive load by narrowing options rather than 
expanding them, and to support task initiation without escalation or 
evaluation pressure.

This orientation comes from direct, ongoing use, including regular 
testing of agent behavior across different underlying models. This repo 
doubles as a working environment for on-going testing.

## Privacy & Data Handling

User profile documents in this system are designed to hold meaningful 
personal context, which means they can include sensitive information. 
Please consider the following before integrating this system into your own workflow:

- The current recommended LibreChat setup stores your profile and log files locally. If you choose to run these agents with local models, Jade Path will never send your data to third parties.
- If you connect these agents to a third-party cloud LLM provider, be aware that data from your user profile and logs, in whole or in part, may be sent to that provider for processing. Review that 
  provider's data handling policies before sending sensitive content. 
  For example, OpenRouter has docs on [Data Collection](https://openrouter.ai/docs/guides/privacy/data-collection) and [Provider Logging](https://openrouter.ai/docs/guides/privacy/provider-logging) that are worth 
  understanding before use.
- Jade Path can easily be adapted to work in web-based environments (claude.ai, Perplexity, etc.) and use MCP connectors (Notion, Dropbox) for external storage of your supporting documents. Please review privacy policies and terms of use carefully for each service involved in request handling.
- Consider what level of detail actually needs to live in a profile 
  document. Specificity helps the agent, but it's worth being 
  deliberate rather than exhaustive.

## Near-term Roadmap

- Complete `memory-architecture.md` and `infrastructure.md`.
- Commit results of ongoing model testing across agents (current: Qwen model efficacy testing with Erika).
