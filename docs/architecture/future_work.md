# Future Work – Pipeline Extensibility

This document outlines planned or potential extensions to the current architecture, focused on making the pipeline modular, dynamic, and extensible.

---

## 1. Dynamic Steps via YAML

Rather than hardcoding the sequence (`train`, `evaluate`, `visualize`), the pipeline could support arbitrary steps declared in `experiment_pipeline.yml`:

```yaml
steps:
  - train
  - evaluate
  - visualize
  - export_summary
  - upload_results
````

Each step would be implemented as a registered module, script, or container.

---

## 2. Step Types: Built-in, Script, Container

Support multiple execution strategies:

| Type        | Description                            |
| ----------- | -------------------------------------- |
| `builtin`   | Python class/function already imported |
| `script`    | External `.py` or `.sh` file           |
| `container` | Docker image with an entrypoint        |

Example:

```yaml
- step: export_summary
  type: script
  path: ./scripts/export.py
```

---

## 3. Distributed Postprocessing

Currently, only training is distributed. Future versions could offload:

* Evaluation to dedicated evaluators
* Visualization to GPU nodes or workers

---

## 4. Event-Driven Pipelines

Replace linear sequencing with an event-driven flow:

```plaintext
[fold_0 DONE] ─┐
[fold_1 DONE] ─┼──> trigger evaluate_results.py
[fold_2 DONE] ─┘
```

This could be implemented via pub/sub systems (e.g., Redis, MQTT).

---

## 5. Plugin System

Support for third-party contributions:

* Each step can be added via a plugin interface
* `run_pipeline.py` discovers them at runtime

---

## Benefits

* More scalable
* Easier to maintain
* Easier to reuse for new workflows (e.g., regression, clustering)
* Flexible enough for external researchers

---

## Status

These extensions are **not required for current experimental goals**, but are fully compatible with the system’s structure and planned for future versions.
