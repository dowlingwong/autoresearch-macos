# Implementation Progress

## Current phase

We are in the first real control-plane phase:

- local Ollama worker exists in the external harness
- structured worker API exists
- project-specific `autoresearch` adapter now owns baseline, state, and keep/discard flow

## Done

- local worker can read/search/edit/run commands
- local worker supports structured task packets
- local worker has machine-readable state and result reporting
- worker lifecycle commands exist: create, run, status, resume, close
- project-specific setup and log parsing commands exist
- automatic baseline handling exists
- pending-candidate state exists
- automatic keep/discard git actions exist
- loop controller with retry rules exists
- experiment memory exists via `experiment_memory.jsonl`
- branch isolation command exists via `autoresearch isolate`
- deterministic control-plane smoke notebook exists
- notebook smoke flow now starts fresh branches from `master`
- notebook smoke flow now requires a successful `train.py` mutation before a run can complete
- edit-required agent tasks now reject wrong-file or failed edits instead of silently succeeding

## In progress

- live real-hardware proof that repeated runs keep `results.tsv` and git state correct
- stronger memory summarization over the raw JSONL event log
- better loop policies for retry, near-miss ideas, and crash clustering
- better prompts and fallback behavior for local models that still drift on bounded edit tasks

## Next

1. prove keep/discard behavior with a few live end-to-end runs on the real repo
2. add worktree isolation for concurrent lanes
3. add richer project memory for experiment history and promising ideas
4. add multi-worker scheduling
5. add k8s environment orchestration
6. add lane-level eventing and monitoring

## Recently resolved issues

- the smoke notebook originally timed out during `baseline` because it used the real training path; this was replaced with a synthetic log writer for control-plane validation
- the worker originally passed edit-required tasks after read-only or wrong-file tool usage; the harness now requires a successful mutation of the intended target file
- repeated smoke runs originally became no-ops because the comment mutation was not repeatable; the smoke mutation now increments a counter instead

## Open design questions

- when should keep/discard remain automatic versus manager-reviewed?
- should experiment memory stay JSONL, move to sqlite, or use both?
- when do we switch from branch isolation to worktree isolation by default?
- when we move to k8s, do we treat each run as a short-lived job or a longer-lived worker pod?
