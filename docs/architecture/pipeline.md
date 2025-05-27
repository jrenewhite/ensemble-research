# Pipeline Execution Flow

This document describes the execution flow handled by `run_pipeline.py`, the general-purpose orchestrator for training, evaluation, and visualization.

## Diagram

![Pipeline Execution Flow](../../rendered/diagrams/PipelineExecutionFlow.svg)

## Description

The pipeline consists of four main phases:

1. **Train**:
   - For each dataset, the runner (monolithic or distributed) is invoked.
   - Each fold is processed, with fold-specific outputs saved and status updated.

2. **Transfer**:
   - In distributed setups, worker results are moved to a shared location or transmitted to the coordinator.
   - Transfer success is recorded per fold.

3. **Evaluate**:
   - Once all folds are marked as `DONE`, the evaluator computes metrics, confusion matrices, and summary statistics.

4. **Visualize**:
   - A separate script transforms evaluation outputs into visual representations.

## Status Tracking

- `status/<dataset>/status.json`: Tracks overall phase completion.
- `status/<dataset>/fold_<i>.json`: Tracks progress and metadata per fold.
- `pipeline_status.json`: Global summary of the execution state.

## Resilience

The pipeline can resume from interruptions by re-reading the `status/` directory and skipping completed steps.
