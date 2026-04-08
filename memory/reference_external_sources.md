---
name: External references
description: External sources, tools, or future systems that are relevant to the autoresearch memory and orchestration roadmap.
type: reference
---

Reference: [`LLM-wiki.md`](/Users/wongdowling/Desktop/LLM-wiki.md)

Why:

- describes a persistent wiki pattern that is a better fit for long-horizon synthesis than plain RAG over raw documents

How to apply:

- use it as the conceptual model for the long-term synthesized memory layer
- do not confuse this with exact run bookkeeping

Reference: [`memory-claude-code.ipynb`](/Users/wongdowling/Downloads/memory-claude-code.ipynb)

Why:

- lays out a practical memory taxonomy and retrieval/injection pattern:
  - `user`
  - `feedback`
  - `project`
  - `reference`

How to apply:

- use the same broad split for future memory growth
- keep this project’s current seed files mostly in `project` and `reference`

Reference: future structured store

Why:

- long-term multi-lane experimentation will need exact querying across runs, branches, and outcomes

How to apply:

- prefer SQLite when the system needs:
  - branch-aware or worktree-aware run history
  - fast filtering across experiments
  - robust analytics over many runs
- keep markdown memory as the maintained synthesis layer even after a structured store is added
