---
name: Failed ideas
description: Failed or cautionary patterns for the autoresearch system, especially around control-plane validation and worker behavior.
type: project
---

Rule: do not treat raw chat output or read-only inspection as evidence that an experiment step succeeded.

Why:

- earlier smoke runs passed superficially while the worker had not actually mutated `train.py`
- the local model sometimes drifted into reading files or attempting the wrong-file edit

How to apply:

- require successful mutation of the intended target file before considering an edit-required task complete
- reject wrong-file edits as failure
- do not create a pending experiment unless `train.py` was actually changed

Rule: do not use the full training command as the default smoke-test path.

Why:

- a smoke test should validate manager/worker control flow quickly
- the real training path is too slow and noisy for basic control-plane validation

How to apply:

- use synthetic `run.log` generation in notebook smoke tests
- reserve real `uv run train.py` runs for live validation and benchmarking

Rule: do not rely on one-shot smoke mutations that become no-ops on later runs.

Why:

- a static comment insertion caused repeated notebook runs to stop producing pending experiments

How to apply:

- use repeatable mutations such as incrementing a smoke-test counter
- explicitly fail smoke tests when the loop result is `no_pending_experiment`
