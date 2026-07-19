# Memory Architecture

This document traces how Jade Path's persistent memory format evolved, the tradeoffs discovered at each stage, and an open problem in how that memory is retrieved once it outgrows a single context window.

## Format Evolution

Memory architecture began with an early version of the Cara agent, in an effort to carry conversational memory forward while keeping token input costs low.

### Stage 1: Prose Summaries

The first approach had the model summarize each chat in plain prose at the end of a session, to be re-fed to the model at the start of the next one. Effective, but not at scale: prose summaries tend to be more verbose than necessary, which costs tokens both at write time and every time the context has to be ingested. Prose is also prone to duplication — the same fact restated slightly differently across summaries, or even within one summary, especially without disciplined prompting.

### Stage 2: JSON Key-Value Pairs

In an attempt to be especially token efficient, the format swung to structured JSON key-value pairs. The tradeoff showed up on the human side: JSON is not friendly to scan when troubleshooting a memory issue or doing manual pruning. Early iterations of Cara also involved working closely with the memory architecture, and editing it by hand was slower and more error-prone than working with structured prose.

### Stage 3: Markdown (Current)

Markdown settled as the middle ground. It is readable for a human doing manual review or edits, while still structured enough (via headings and grouped sections) to stay LLM-friendly.

Markdown's advantage extends beyond the write side. Once a memory log grows past what fits in full context and needs to be stored externally and searched (via RAG) rather than loaded wholesale, structured Markdown headings give a chunking algorithm clean, predictable boundaries to split on — a property flat prose and JSON don't offer as cleanly, since JSON's structure is nested and syntactic rather than semantic.

A version of Cara's session-log recording format reflects this directly:

```markdown
## [Session Date] — [Summary Title]
Session type: [a few keywords] Mood/state: [one-sentence summary]

### What happened

### Flags for next session
```

This schema does double duty: it gives the retrieval layer a consistent chunking boundary per entry, and it functions as an schema/example that keeps the writing model's output consistent across sessions.

## Open Problem: RAG Retrieval Tuning

The chosen memory format (Markdown) is settled, but how that memory is retrieved once it exceeds full-context size is not. LibreChat's built-in file search (a self-hosted rag_api service backed by pgvector) is configuration-driven rather than a managed black box as it would be on a commercial web app. Chunk size, chunk overlap, embedding model, and retrieval count are all exposed as tunable settings rather than auto-optimized. This is a meaningful difference from Perplexity's and claude.ai's memory handling, which likely have highly-iterated retrieval management processes.

Recommended LibreChat settings for file search are scheduled for future work. Furthermore, LibreChat's file management is a current active focus of development, which this project will continue to monitor.

## Roadmap

- Tune LibreChat rag_api chunk size/overlap against the `session-log` corpus.
- Evaluate whether embedding model choice affects retrieval quality meaningfully at this corpus size.
