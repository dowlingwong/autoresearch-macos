# Smoke Test

Use these commands from [`/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code).

There are two valid smoke-test modes:

- deterministic control-plane smoke test
- deterministic memory smoke test
- live end-to-end training smoke test

The deterministic path is the recommended first check because it validates the manager/worker control flow quickly without waiting for a full train run.

## Recommended notebook smoke test

Open [`autoresearch_control_plane_demo.ipynb`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code/notebooks/autoresearch_control_plane_demo.ipynb) from [`harness/claw-code`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code).

What it validates:

- `setup` and `status`
- isolated branch creation from `master`
- baseline recording
- worker mutation of `train.py`
- pending experiment creation
- `keep` or `discard`
- one `loop` iteration

Important:

- this notebook uses a synthetic `run.log` writer instead of a real training run
- it is a control-plane validation, not a benchmark

## Recommended memory smoke test

Open [`autoresearch_memory_smoke_test.ipynb`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code/notebooks/autoresearch_memory_smoke_test.ipynb) from [`harness/claw-code`](/Users/wongdowling/Documents/autoresearch_harness/harness/claw-code).

What it validates:

- the `memory/` directory exists
- `memory/MEMORY.md` links the expected topic files
- each topic file contains valid frontmatter
- `results.tsv` still exists as the exact ledger
- `experiment_memory.jsonl` still exists as the append-only event log

Important:

- this notebook validates the memory layout, not experiment quality
- it is meant to confirm the current hybrid memory design before adding SQLite or richer retrieval

## 1. Verify project setup

```bash
python3 -m src.main autoresearch setup --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos
python3 -m src.main autoresearch status --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos
```

Expected:

- `results.tsv` exists
- `.autoresearch_state.json` exists
- `data_ready` and `tokenizer_ready` match the local cache state

## 2. Create an isolated branch

```bash
python3 -m src.main autoresearch isolate --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos --branch autoresearch/demo --create
```

Expected:

- current branch becomes `autoresearch/demo`

## 3. Record the baseline

```bash
python3 -m src.main autoresearch baseline --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos
```

Expected:

- `baseline_created` is `true` the first time
- `results.tsv` gets a `keep	baseline` row

## 4. Run one candidate

```bash
python3 -m src.main autoresearch run --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos --packet /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos/experiment.demo.json --trace
python3 -m src.main autoresearch status --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos
```

Expected:

- worker result contains non-empty `tool_calls`
- `changed_files` includes `train.py`
- state shows a non-null `pending_experiment`
- `recommended_status` is `keep`, `discard`, or `crash`

## 5. Finalize the candidate

If the candidate should advance:

```bash
python3 -m src.main autoresearch keep --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos
```

If the candidate should be reverted:

```bash
python3 -m src.main autoresearch discard --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos
```

Expected:

- `results.tsv` gets a new row
- `pending_experiment` becomes `null`
- discard resets git back to the candidate's base commit

## 6. Run an automated loop

```bash
python3 -m src.main autoresearch loop --root /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos --packet /Users/wongdowling/Documents/autoresearch_harness/nodes/autoresearch-macos/experiment.demo.json --iterations 2 --retry-limit 1 --trace
```

Expected:

- baseline is ensured automatically
- each history entry includes `run` and `decision`
- `experiment_memory.jsonl` grows after each stage
- loop decision is not `no_pending_experiment`

## Real proof checklist

The local agent side is only "circle-ready" after a few real runs show:

- `results.tsv` rows are appended correctly
- `keep` leaves the new commit in place
- `discard` resets back to the previous kept commit
- crash runs are logged as `crash`
- repeated loop iterations leave the repo and state files consistent
- the real `uv run train.py` path behaves well across several consecutive iterations, not just the synthetic smoke path
- the synthesized topic memory stays coherent and useful as exact run history grows
