# Execution Flow – Monolithic and Distributed

This document explains the complete execution flow of the system under both **monolithic** and **distributed** modes. It complements the sequence diagrams found in `docs/architecture/`.

---

## Controlled by `run_pipeline.py`

All execution is orchestrated by the `run_pipeline.py` script, which reads an `experiment_pipeline.yml` file and runs:

1. **Training**
2. **Transfer (if needed)**
3. **Evaluation**
4. **Visualization**

Each step's status is tracked via JSON files inside the `status/` directory.

---

## Monolithic Flow (`main_monolithic.py`)

### Overview

- Executed per dataset and K-Fold value.
- Uses a single machine, sequential folds.
- Produces predictions and probabilities per fold.
- Defers evaluation and visualization to later steps.

### Flow Summary

1. `run_pipeline.py` checks if training is needed for the dataset.
2. It launches `main_monolithic.py` with:
   - Dataset config path
   - Output directory
   - `--kfold` value
3. For each fold:
   - The fold status is checked (`status/<dataset>/fold_<i>.json`).
   - If not done, training is performed.
   - Outputs are saved: predictions, probabilities, rankings, metadata.
   - Fold is marked as `DONE`.
4. Once all folds are complete:
   - `train_status = DONE` is recorded.
   - Evaluation and visualization steps can begin.

📎 See: [Monolithic Sequence Diagram](../architecture/sequence_monolithic_execution.puml)

---

## Distributed Flow (`main.py --role=worker|coordinator`)

### Overview

- Multiple machines can participate.
- Coordinator assigns folds to workers.
- Each worker handles training for a fold and sends results back.
- Status is updated centrally.

### Flow Summary

1. `run_pipeline.py` starts the coordinator.
2. Coordinator reads pending folds from `status/`.
3. Workers connect and request tasks.
4. Coordinator sends:
   - `job.json`, `system.json`
   - Fold ID
5. Each worker:
   - Loads the dataset
   - Trains its assigned fold
   - Saves outputs locally
   - Sends an ACK (or results) back to the coordinator
6. Coordinator marks the fold as `DONE`.

When all folds are `DONE`, training is considered complete, and evaluation can begin.

📎 See: [Distributed Sequence Diagram](../architecture/sequence_distributed_execution_detailed.puml)

---

## Output Structure

After training, results are stored under:

```

results/<dataset>/fold\_<i>/
├── predictions.csv
├── probabilities.csv
├── ranks.csv
├── metadata.json
└── .ready (optional marker)

```

Status is tracked here:

```

status/<dataset>/fold\_<i>.json
status/<dataset>/status.json

```

---

## Evaluation Phase

Once all folds are `DONE`:

- `run_pipeline.py` runs `evaluate_results.py`
- It computes metrics from the fold outputs
- The result is a summary JSON + CSV

Example outputs:

```

results/<dataset>/
├── evaluation\_summary.json
├── evaluation\_summary.csv

```

---

## Visualization Phase

After evaluation:

- `generate_visuals.py` generates plots or animations
- Based on `evaluation_summary.json`

---

## Reentrancy & Resilience

The system supports recovery:

- If a fold fails, it can be retried independently.
- If a run is interrupted, `run_pipeline.py` skips already-completed steps.
- Workers or coordinators can crash and restart without invalidating the pipeline.

---

_Last updated: 2025-05-25_
