---
name: Promising ideas
description: Promising next steps for memory, orchestration, and experiment quality in autoresearch.
type: project
---

Direction: keep the current hybrid memory model.

Why:

- `results.tsv` is good for exact run tracking
- `experiment_memory.jsonl` is good for append-only event history
- topic markdown files are good for accumulated strategy and retrieval

How to apply:

- continue using the topic memory directory for synthesized knowledge
- move to a structured store only when query complexity or concurrency requires it

Direction: make the next memory layer branch-aware or lane-aware.

Why:

- current best/baseline state carries across branches
- this is manageable for a single lane, but will be confusing for parallel lanes

How to apply:

- add branch/worktree identifiers to future structured memory records
- avoid assuming repo-global state maps cleanly to multi-lane operation

Direction: add a structured store later rather than now.

Why:

- the current system still needs repeated real-run validation
- SQLite becomes most useful when there are many runs, many branches, or many workers

How to apply:

- keep the near-term memory lightweight
- add SQLite after worktree isolation and multi-lane scheduling become real requirements

Direction: use the manager to maintain a living research wiki.

Why:

- long-term strategy benefits from persistent synthesis, not just retrieval
- the LLM wiki pattern compounds over time and reduces repeated rediscovery

How to apply:

- use topic files for strategy, failures, promising hypotheses, and references
- update existing files before creating new ones
