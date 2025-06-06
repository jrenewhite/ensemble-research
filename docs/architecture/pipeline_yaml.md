# Experiment Pipeline YAML Specification

This document describes the structure and semantics of the `experiment_pipeline.yml` configuration file used to define and control the execution of ensemble learning experiments within the system.

---

## Experiment Metadata

### `experiment`

Metadata describing the current experiment.

```yaml
experiment:
  id: iris_benchmark_01  # Must match ^[a-zA-Z0-9_\-]{3,32}$
  name: "Iris Classification Benchmark"
  description: "Test with RF, SVM, NB, LR on Iris dataset"
  author: "José René White Enciso"
  date: "2025-06-03"
```

* **id**: Unique identifier for the experiment. If omitted, a hash-based ID will be generated.

  * Must match: `^[a-zA-Z0-9_\-]{3,32}$`
* **name**: Optional name to help identify the experiment.
* **description**: Longer explanation of the purpose.
* **author**: Person or system defining the experiment.
* **date**: ISO date string when this configuration was generated.

---

## Global Configuration

### `global`

Top-level default values applied to all datasets unless overridden individually.

```yaml
global:
  output_root: ./results
  kfold: 5
  random_seed: 42
  default_split_ratio: 0.2
  base_models: [rf, svm, lr, nb]
  ensemble_size: 3
  num_ensembles: 2
  log_level: INFO
```

* **output\_root**: Path to the root directory where results are saved.
* **kfold**: Number of folds for cross-validation.
* **random\_seed**: Seed to ensure reproducibility.
* **default\_split\_ratio**: Used if a dataset doesn’t have its own split ratio.
* **base\_models**: List of base learners to use.
* **ensemble\_size**: How many base models are combined into one ensemble.
* **num\_ensembles**: Number of ensembles to train.
* **log\_level**: Logging verbosity (DEBUG, INFO, WARN, ERROR).

---

## Execution Modes

### `modes`

Specify which engine types to run.

```yaml
modes:
  monolithic: true
  distributed: true
```

* **monolithic**: Runs local experiments.
* **distributed**: Enables multi-node coordination.

---

## Role Permissions

### `roles`

Defines what roles can promote to others.

```yaml
roles:
  experimenter:
    allow_be_coordinator: true
    allow_be_worker: false
  coordinator:
    allow_be_worker: true
```

* **allow\_be\_coordinator**: Whether a node may promote itself to coordinator.
* **allow\_be\_worker**: Whether this role can perform worker tasks (training, etc.).

---

## Execution Steps

### `steps`

Control which stages of the pipeline to run.

```yaml
steps:
  train:
    enabled: true
    retry_failed: true
    max_time_per_fold: 600

  evaluate:
    enabled: true
    strategy: [soft_vote, probs, rank]
    metrics: [accuracy, f1_macro, precision_macro, recall_macro]

  visualize:
    enabled: true
    renderers: [motion_canvas, matplotlib]
    export_video: true

  statistics:
    enabled: true
    tests: [anova, tukey]
    compare_ensembles_only: true

  transmit:
    enabled: true
    what: [predictions, model]
    method: auto

  inference:
    enabled: false
    strategy: soft_vote
    refresh_models_on_change: true
```

* **train.max\_time\_per\_fold**: Timeout in seconds for each fold. If exceeded, node is marked failed.
* **evaluate.strategy**: Strategies to use when aggregating ensemble predictions.
* **visualize.renderers**: Multiple renderers allowed (e.g., motion\_canvas, matplotlib).
* **statistics.tests**: Statistical comparisons to apply.
* **transmit.what**: What artifacts to send (`predictions`, `model`, `logs`).
* **transmit.method**: Communication strategy (`auto`, `push`, `pull`).
* **inference.strategy**: How to respond in real-time inference mode.

---

## Dataset Configurations

### `datasets`

List of datasets with optional overrides.

```yaml
datasets:
  - name: iris
    config_path: ./data/configurations/iris
    override:
      split_ratio: 0.25
      kfold: 3
      output_root: ./results/iris
      base_models: [rf, nb, lr]
      ensemble_size: 2
      num_ensembles: 4
      roles:
        experimenter:
          allow_be_coordinator: true
          allow_be_worker: true
```

* **override**: Any field from `global`, `modes`, `roles`, or `steps` can be customized here.

---

## System Integration

### Generation of Legacy Configs

```yaml
generation:
  job_spec: true
  system_spec: true
  overwrite_existing: false
```

* **job\_spec**: Generate `job.json`.
* **system\_spec**: Generate `system.json`.
* **overwrite\_existing**: If false, keep existing config files unless explicitly deleted.

---

## Notes on Execution

* The experimenter node reads this YAML and organizes the workflow accordingly.
* Each dataset is evaluated independently, using roles and steps as dictated.
* If a dataset is already trained, evaluation can continue.
* Ensembles are identified using unique hashes and human-friendly names (e.g., "343 Iris Singing Sparrow").
* Nodes may retry tasks that time out or fail.
* Statistics help validate the effectiveness of model ensembles.
* Transmit step ensures necessary outputs reach the right components.
* Inference step allows for prediction service continuation after training.

---

## Example Renderers

* `motion_canvas`: For dynamic animations.
* `matplotlib`: For static charts.

---

## Example Strategies

* `soft_vote`: Majority class prediction based on probabilities.
* `probs`: Use raw class probabilities.
* `rank`: Use ranking information from models.

---

## Example Statistical Tests

* `anova`: One-way analysis of variance.
* `tukey`: Tukey’s HSD post-hoc test.

---

## Final Considerations

This YAML file is the **single source of truth** for experiment configuration and allows high reproducibility, resilience, and extensibility. Each part of the system reads only the portions it needs and performs the step it is responsible for.
