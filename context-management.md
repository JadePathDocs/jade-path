# Context Management

Long-running AI agents accumulate memory over time: session logs, identity documents, progress trackers, study plans. Left unmanaged, these files grow past what's practical to load in full on every interaction, creating a tension between "the agent remembers everything" and "the agent stays fast, cheap, and accurate." Between pricing for LLM use being dependent on input size and hard limits on how much input there can be in one request, these are important considerations for any use case.

This document covers the context budget decisions made across Jade Path's agents, the tradeoffs behind them, and where the unsolved edges still are.

## Growth Patterns Observed

Not every supporting file grows at the same rate, and file type predicts growth more reliably than agent identity:

- **Static identity documents** (agent role, rules, corrections) stay small. One tested agent's identity file sits at roughly 2,300 characters after an extended period of testing. Updates are infrequent and additive only when a rule is corrected twice.
- **Structured progress/study logs** can grow faster than expected. User testing on a comparable log structure for the Erika agent suggests logs in the 15,000-20,000 character range are reachable within 2-3 months of consistent daily use, even with a deliberately minimal schema.
- **Pre-built plan documents** may start large. A 32-week structured Erika study plan landed near 20,000 characters at creation, before any usage-driven growth begins.
- **Rolling session logs** grow linearly with usage frequency. One agent's session log is projected to exceed ~100k characters after three months of regular use. Depending on cost considerations and infrastructure accommodations, this is already well past the point of economic full-context usage and requires a file search solution.

As observed, growth rate depends heavily on document type, not just time elapsed. A cost-effective implementation gives a rolling log and a static reference file different context strategies from day one, not just different strategies once they get large.

## Platform Behavior (Observed, Not Documented)

Context handling differs across platforms and actual behavior is not always fully documented by the provider.

- **Claude.ai**: Large available context window, but full-context loading at scale is a cost-per-token problem rather than a hard wall. Cost scales with what's actually loaded, not just what's theoretically available.
- **Perplexity**: Documented context is smaller than Claude's, and no hard limit is stated, but heavy use in long chats evinces a rolling context window. Content loaded early in a session (like an agent's memory file at the start of a conversation) is the first to be dropped as the conversation grows. Observed effective capacity in practice appears smaller than the documented 32k token context window, suggesting some degree of overhead not apparent to the user.
- **Self-hosted RAG (Open WebUI, LibreChat)**: For extracting small bursts of context from large documents, retrieval quality on self-hosted front ends has, in practice, been noticeably weaker than either commercial platform's built-in handling. The exact cause (chunking strategy, embedding choice, retrieval parameters) has not yet been isolated.

Note that these are behavioral observations from regular use, not benchmarked measurements. Testing this formally would require controlled experiments outside the current project scope.

## Current Approach

Given the above, Jade Path currently avoids solving general-purpose retrieval and instead scopes context per agent:

- Agents likely to need only a handful of small, stable reference files are given full-context loading as entire files included in prompts.
- Files that grow unpredictably (i.e., rolling session logs) get trimmed manually or resummarized by LLM when they start impacting response quality, rather than automatically chunked or embedded.
- New supporting files added to an agent are evaluated against this same context budget lens before being added. The working question is always "does this file's growth pattern fit full-context loading, or will it need a different strategy soon?"

A RAG vector search-based strategy is the most valuable next investment area, especially for Cara's session log.

## Known Limitation and Considerations for Future Agent Development

This approach is a work in progress, appropriate for this early stage. It works because current agents have a small number of files with mostly predictable growth. It will not scale to agents with many large, fast-growing files, or to any use case requiring search across a large document set. That use case would require formal retrieval evaluation, specifically measuring retrieval accuracy against chunk size and embedding choice.
