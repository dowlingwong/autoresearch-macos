# Agent System

This project is being adapted to run under a manager/worker research harness.

## Core idea

- A stronger manager agent such as Codex or Claude Code handles planning, prioritization, experiment bookkeeping, and branch policy.
- A local worker agent such as `qwen2.5-coder` over Ollama handles bounded implementation tasks inside the repo.
- The repo-specific experiment runner remains the source of truth for training/evaluation outcomes.

## Why this split

`autoresearch-macos` is not a normal application repo.

Its core loop is:

1. modify `train.py`
2. run a fixed-time experiment
3. parse the resulting metric
4. keep or discard the attempt

That means the worker should not own strategy. It should own bounded execution. The manager should own experiment policy and long-horizon reasoning.

## Immediate architecture

- `program.md` remains the high-level research spec
- `prepare.py` remains read-only infrastructure
- `train.py` remains the only mutable research surface
- the external harness provides a deterministic control plane:
  - worker task packets
  - worker state/status
  - branch isolation
  - baseline recording
  - structured experiment runs
  - keep/discard actions
  - result logging
  - experiment memory

## Current control-plane files

The project root now acts as a stable handoff point between the manager and the local worker:

- `results.tsv`: durable experiment ledger
- `.autoresearch_state.json`: current branch, best result, and pending candidate state
- `experiment_memory.jsonl`: append-only event log for baseline, candidate, keep, discard, and loop events
- `memory/`: synthesized topic memory for current strategy, failed ideas, promising ideas, and external references

## Current control-plane commands

The external harness now exposes deterministic project commands:

- `autoresearch setup`
- `autoresearch status`
- `autoresearch isolate`
- `autoresearch baseline`
- `autoresearch run`
- `autoresearch keep`
- `autoresearch discard`
- `autoresearch loop`

## Current smoke-test path

There are now two distinct validation paths:

- a deterministic control-plane smoke test in [`harness/claw-code/notebooks/autoresearch_control_plane_demo.ipynb`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code/notebooks/autoresearch_control_plane_demo.ipynb)
- a live project run against the real training command and real Ollama worker

The notebook path is intentionally cheaper and faster:

- it creates an isolated branch from `master`
- it uses a synthetic `run.log` writer instead of a full training run
- it verifies `setup`, `status`, `isolate`, `baseline`, `run`, `keep/discard`, and `loop`
- it enforces that the worker must successfully mutate `train.py`, not just inspect files

This is useful for validating the manager/worker control plane before spending time on real training runs.

There is now a separate memory smoke notebook in [`harness/claw-code/notebooks/autoresearch_memory_smoke_test.ipynb`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code/notebooks/autoresearch_memory_smoke_test.ipynb) that validates the synthesized memory layer.

## Future architecture

Later phases should add:

- worktree-backed isolation for concurrent lanes
- multiple workers per repo
- richer memory for promising ideas, failed directions, and run history
- Kubernetes-backed environment orchestration for isolated experiment lanes
- typed event streams for worker state and experiment outcomes

## Current status

The current target is still a single-worker adapter, not a full swarm.

That adapter now can:

- ask the worker to modify only `train.py`
- record a baseline automatically
- run one experiment
- parse `run.log`
- hold a pending candidate until a keep/discard decision is made
- update `results.tsv`
- record memory events
- report machine-readable state back to the manager

The main unresolved gap is no longer control-plane wiring. It is proving that repeated real runs keep git state, `results.tsv`, and long-horizon experiment quality stable over time.

The current memory strategy is:

- short-term exact records: `results.tsv`
- append-only operational history: `experiment_memory.jsonl`
- synthesized project memory: `memory/*.md`

The planned long-term enhancement is:

- SQLite or another structured store for exact querying across many runs and lanes
- maintained markdown memory/wiki for long-horizon synthesis
- selective retrieval over both layers when needed
