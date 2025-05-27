# Evaluation and Visualization Modules

This document describes the components involved in analyzing and visualizing the results of ensemble learning experiments after all folds have been trained. These are located in the `src/evaluation/` directory and are invoked independently by `run_pipeline.py`.

---

## 1. evaluate_results.py

### Purpose

Aggregates the raw output of each fold to compute performance metrics. These include standard classification scores, confusion matrices, and statistical summaries.

### Responsibilities

- Read fold outputs:
  - `predictions.csv`
  - `probabilities.csv`
  - `metadata.json`
- Compare predictions to true labels.
- Compute:
  - Accuracy
  - Precision
  - Recall
  - F1-score
  - Confusion matrix
- Export evaluation summaries:
  - `evaluation_summary.json`
  - `evaluation_summary.csv`
  - (Optional) confusion matrices as PNGs

### Inputs

- Path to `results/<dataset>/`
- Fold outputs per fold

### Outputs

- `evaluation_summary.json`: Machine-readable aggregate metrics
- `evaluation_summary.csv`: Tabular representation of the above
- Optional: `confusion_matrix_<fold>.png` (if rendering is enabled)

---

## 2. generate_visuals.py

### Purpose

Creates visual representations of evaluation results. May include static charts or animations depending on context (e.g., Motion Canvas integration).

### Responsibilities

- Load:
  - `evaluation_summary.json`
- Generate:
  - Confusion matrix plots
  - Accuracy bar charts
  - Latency/performance graphs
  - Optional: Export structured data for animation

### Inputs

- `evaluation_summary.json`

### Outputs

- `plots/` folder within results:
  - `accuracy_by_model.png`
  - `confusion_matrix_all.png`
  - `ensemble_timing.png`
- Optional: `visuals.json` (intermediate data for external rendering)

---

## Component Diagram

![Evaluation and Visualization Components](../../rendered/diagrams/evaluation_modules.svg)

---

## Execution Flow

The following illustrates when each module is triggered:

```plaintext
run_pipeline.py
     └── evaluate_results.py   → after all folds are DONE
           └── generate_visuals.py → after evaluation is complete
````

---

## Example

```bash
# Evaluate results from trained folds
python evaluate_results.py --input ./results/iris

# Generate visuals from summary
python generate_visuals.py --input ./results/iris/evaluation_summary.json
```

