# Hierarchy

## Roles

### 1. Manager agent

Examples:

- Codex
- Claude Code

Responsibilities:

- understand `program.md`
- propose experiment direction
- decide whether a run is worth keeping
- manage experiment sequencing
- manage branch policy and repo hygiene
- aggregate logs, results, and memory
- maintain the synthesized project memory in `memory/`
- decide when to move from branch isolation to worktree or k8s lane isolation

### 2. Worker agent

Example:

- local Ollama model such as `qwen2.5-coder`

Responsibilities:

- read the in-scope files
- modify only `train.py`
- perform bounded code changes
- run allowed local tools
- return structured change summaries

### 3. Project runner

Responsibilities:

- verify repo setup
- isolate or validate the experiment branch
- ensure a baseline exists
- run `uv run train.py`
- enforce timeout
- parse metrics from `run.log`
- keep or discard the candidate
- update `results.tsv`
- persist state and experiment memory
- leave higher-level synthesis to the topic memory layer in `memory/`

This runner should be deterministic and project-specific. It should not depend on free-form model output for core experiment bookkeeping.

The current runner also distinguishes between:

- control-plane smoke runs using synthetic logs
- live experiment runs using the real `uv run train.py` path

The current project memory also has two layers:

- exact operational memory:
  - `results.tsv`
  - `.autoresearch_state.json`
  - `experiment_memory.jsonl`
- synthesized topic memory:
  - `memory/MEMORY.md`
  - `memory/project_current_strategy.md`
  - `memory/project_failed_ideas.md`
  - `memory/project_promising_ideas.md`
  - `memory/reference_external_sources.md`

That split exists so the manager/worker protocol can be tested without waiting for a full training cycle.

## Control flow

1. Manager creates a project-specific experiment packet.
2. Project runner verifies branch isolation and baseline state.
3. Worker edits `train.py`.
4. Project runner executes the experiment and records a pending candidate.
5. Project runner or manager finalizes `keep` or `discard`.
6. Manager schedules the next attempt.

For notebook smoke tests, step 4 uses a synthetic log command. For real experiments, step 4 uses the actual training command.

Parallel to that control flow, the manager should periodically fold exact run history into the synthesized topic memory so strategy improves over time instead of being rediscovered from raw logs.

## Future hierarchy

Later, add:

- a swarm coordinator over multiple workers
- a shared experiment memory layer backed by a structured store
- worktree-backed isolation for concurrent lanes
- environment scheduling across local and k8s-backed workers
