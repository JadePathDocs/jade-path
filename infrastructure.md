# Infrastructure

This document covers the self-hosted stack behind Jade Path, the tradeoffs discovered in production-simulated testing, and open problems still being worked through.

## Stack Overview

The current setup runs LibreChat in Docker, with agents (presently Cara, Erika) configured as separate agent instances, each with its own project instructions, pre-selected model, and external memory files. A request path can described roughly thus:

1. A prompt is entered in a LibreChat chat tied to a specific agent.
2. LibreChat bundles the prompt with the agent's project instructions and connected memory file(s) and sends it over API — either directly to a provider (e.g. Anthropic) or through OpenRouter.
3. OpenRouter routes the request to a provider for the selected model, using default load-balancing or configured preferences.
4. The provider runs inference and returns a response through OpenRouter (if used) back to LibreChat.

Docker itself sits outside this request path. Its containers allow the system to run self-hosted rather than requiring a rented server, but it does not directly handle individual requests.

## Provider and Model Selection

As the lead agent for Jade Path, Cara was built with sensitivity and capacity for responding to low-mood sessions in mind. Early testing leaned toward Claude models executing the Cara instructions most effectively and consistently, possibly due to Anthropic's Reinforcement Learning from Human Feedback (RLHF) optimization. Ultimately, the efficacy of Cara's support may come down to individual user preference for a given model, and trying the agent on different models is encouraged for users seeking to cut costs, given Anthropic's current price structure.

Erika (and other, more specialized, task agents) may be more flexible. Testing has shown Erika can run even advanced grammar discrimination tasks on alternative reasoning models such as DeepSeek Pro v4, hosted by OpenRouter for a fraction of the cost of Claude Opus, for example. Claude.ai offers a plug-and-play solution that the Jade Path system can be dropped into, but the lack of ability to switch models mid-chat necessitates task separation or additional memory work, adding friction that may not be the best for the target audience of this project.

## Token Caching: Open WebUI vs. LibreChat

Caching Claude API context (system instructions, memory files) between calls matters a lot at Opus-tier pricing, where an uncached repeated context gets billed in full on every turn. Open WebUI had no native caching support for Claude's API. A community pipeline plugin existed to patch this, but it was unreliable, a potentially expensive failure given the model tier involved. Caching behavior is also notoriously hard to verify in isolation. Cache hits/misses don't always show clearly in a single request; the failure mode tends to surface only across a multi-turn conversation, which made the Open WebUI plugin's reliability difficult to confirm or trust. Testing on this was done ahead of time against Claude Haiku rather than in production, keeping the cost of discovering this problem in the range of quarters rather than dollars.

Adoption of LibreChat solves this natively: caching is a toggle in the agent's settings, configurable at 5-minute or 1-hour TTL, and works consistently whether the request goes over a direct Claude API key or through OpenRouter to another provider.

### Open Problem: Cache Loss on Provider Switching

OpenRouter's automatic routing could, in theory, switch providers mid-conversation, invalidating any active prompt cache tied to the previous provider. This hasn't been observed in practice at this project's usage scale, but the theoretical exposure exists.

OpenRouter supports request-level provider preferences (e.g. provider order, disabling fallbacks) that could pin a conversation to one provider. However, LibreChat's current request formation does not appear to expose a way to pass these preferences through. A workaround, if one exists, would likely need to come from an account-level default set directly in OpenRouter's web UI rather than anything configurable in LibreChat today. This is unresolved and unconfirmed at this point.

## MCP + Docker Networking: Yomitoku Installation Troubleshooting

Erika (the Japanese study agent) potentially needs OCR support for image-based material. This was seamless on claude.ai's native image handling, but became a separate problem when testing cheaper non-Claude models (DeepSeek, Qwen) for the reasoning side of grammar-discrimination tasks, since the cheaper, text-only variants of those models do not handle images.

The fix was to separate concerns: run OCR through a dedicated tool (Yomitoku) rather than requiring the reasoning model itself to handle images. This also served as a first real test of MCP within LibreChat.

Getting the two Docker containers (LibreChat and Yomitoku) to connect took several hours of troubleshooting, surfacing as a persistent 421 error. AI-assisted research uncovered the root cause: MCP versions 1.9.1 and later introduced stricter request validation (DNS-rebinding protection), which checks the Host/Origin header against a trusted allowlist. That allowlist defaults to covering localhost, not the internal hostnames Docker containers use to reach each other on a shared network. Downgrading to MCP 1.9.0, which predates the stricter default, resolved the connection  problem immediately.

### Cost Lesson: Subagents for Tool Calls

A secondary finding from this process: routing a tool call (like an OCR request) directly through the primary reasoning model roughly doubles the cost of that call, since the tool-call overhead is billed at the reasoning model's rate. Using LibreChat's subagent feature to hand tool calls to a lightweight model (a cheap model like DeepSeek Flash, or a local Ollama instance) and passing the result back to the primary agent avoids this — the heavier model is reserved for the actual reasoning task, not for shuttling tool output.
