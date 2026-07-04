# Context Management

## The Problem

Long-running AI agents accumulate memory over time: session logs, identity documents, progress trackers, study plans. Left unmanaged, these files grow past what's practical to load in full on every interaction, creating a tension between "the agent remembers everything" and "the agent stays fast, cheap, and accurate." Between pricing for LLM use being dependent on input size and hard limits on how much input there can be in one request, these are important considerations for any use case.

This document covers the context budget decisions made across Jade Path's agents, the tradeoffs behind them, and where the unsolved edges still are. 

## Growth Patterns Observed

Not every supporting file grows at the same rate, and file type predicts growth more reliably than agent identity:

- **Static identity documents** (agent role, rules, corrections) stay small. One tested agent's identity file sits at roughly 2,300 characters after an extended period of testing. Updates are infrequent and additive only when a rule is corrected twice.
- **Rolling session logs** grow linearly with usage frequency. One agent's session log is projected to exceed ~100k characters after three months of regular use. Depending on cost considerations and infrastructure accommodations, this is already well past the point of economic full-context usage and requires a file search solution.
- **Structured progress/study logs** grow faster than expected even with disciplined entries. User testing on a comparable log structure for the Erika agent suggests logs in the 15,000-20,000 character range are reachable within 2-3 months of consistent daily use, even with a deliberately minimal schema.
- **Pre-built plan documents** can start large. A 32-week structured Erika study plan landed near 20,000 characters at creation, before any usage-driven growth begins.

The practical implication: growth rate depends heavily on document *type*, not just time elapsed. A rolling log and a static reference file need different context strategies from day one, not just different strategies once they get large.

## Platform Behavior (Observed, Not Documented)

Context handling differs across platforms, and in the actual behavior isn't always fully documented by the provider.

- **Claude.ai**: Large available context window, but full-context loading at scale is a cost-per-token problem rather than a hard wall. Cost scales with what's actually loaded, not just what's theoretically available. 
- **Perplexity**: Documented context is smaller than Claude's, and no hard limit is stated, but heavy use in long chats evinces a rolling context window. Content loaded early in a session (like an agent's memory file at the start of a conversation) is the first to be dropped as the conversation grows. Observed effective capacity in practice appears smaller than the documented 32k token context window, suggesting some degree of overhead not apparently visible to the user.
- **Self-hosted RAG (Open WebUI, LibreChat)**: For extracting small bursts of context from large documents, retrieval quality on the self-hosted front ends in practice has been noticeably weaker than either commercial platform's built-in handling, though the exact cause (chunking strategy, embedding choice, retrieval parameters) hasn't been isolated yet.

Note that these are behavioral observations from regular use, not benchmarked measurements. Testing this formally would require controlled experiments outside the current project scope.

## Current Approach

Given the above, Jade Path currently avoids solving general-purpose retrieval and instead scopes context per agent:

- Agents likely to need only a handful of small, stable reference files are given full-context loading:  the whole file, every time, no retrieval step.
- Files that grow unpredictably (like rolling session logs) get trimmed manually or resummarized by LLM when they start impacting response quality, rather than automatically chunked or embedded.
- New supporting files added to an agent are evaluated against this same context budget lens before being added. The working question is always "does this file's growth pattern fit full-context loading, or will it need a different strategy soon?"

A RAG vector search-based strategy should be a top priority for further development of this project, especially for Cara's session log.

## Known Limitation and Considerations for Future Agent Development

This approach is a scoped tradeoff, not a solved problem. It works because current agents have a small number of files with mostly predictable growth. It will not scale to agents with many large, fast-growing files, or to any use case requiring search across a large document set. Solving that properly would mean formal retrieval evaluation, specifically measuring retrieval accuracy against chunk size and embedding choice.
