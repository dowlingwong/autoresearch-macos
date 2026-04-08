---
name: Current strategy
description: Current operating strategy for the autoresearch manager/worker system and how to use the control plane safely.
type: project
---

Use a layered memory design:

- exact run bookkeeping in `results.tsv`
- append-only event history in `experiment_memory.jsonl`
- synthesized topic memory in `memory/*.md`

Current operating mode:

- single-lane branch-isolated experimentation
- manager/worker split:
  - manager handles planning, bookkeeping, and keep/discard policy
  - local Ollama worker handles bounded `train.py` edits
- deterministic control-plane commands:
  - `setup`
  - `status`
  - `isolate`
  - `baseline`
  - `run`
  - `keep`
  - `discard`
  - `loop`

Current priorities:

- prove repeated live runs with the real `uv run train.py` path
- keep `train.py` as the only writable experiment surface
- treat branch isolation as sufficient for one lane, but not for multi-worker concurrency

Why:

- raw logs alone are too noisy for planning
- generic RAG over code and logs does not accumulate strategy well
- a maintained topic memory can store hypotheses, constraints, and lessons learned across runs

How to apply:

- update this file when the control-plane shape or operating mode changes
- keep exact metrics and per-run outcomes in `results.tsv`
- only promote stable conclusions here after they have been observed across runs or validated by the manager
