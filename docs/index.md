# Ensemble Learning Research System

Welcome to the documentation for the Ensemble Learning Research System. This project supports both monolithic and distributed execution of ensemble learning experiments, focusing on classification tasks across a wide range of datasets. The system is designed for modularity, reproducibility, and in-depth evaluation of results.

---

## Overview

This system includes:

- A **monolithic engine** for local, single-machine execution with full K-Fold support.
- A **distributed engine** using coordinator/worker roles across networked devices (e.g., SBC clusters, old machines).
- A **pipeline orchestrator (`run_pipeline.py`)** that automates end-to-end workflows using a YAML descriptor.
- An **evaluation and visualization subsystem** that runs independently after training is complete.

---

## Execution Flow

Execution is organized into four main phases:

1. **Train**: Models are trained fold by fold.
2. **Transfer**: Fold results are transmitted to a central evaluation node (if distributed).
3. **Evaluate**: Metrics are calculated from prediction outputs.
4. **Visualize**: Final charts or animations are produced from summary data.

Each step writes its state to the `status/` directory to support recovery and traceability.

---

## Experiments

Performed experiments include:

- **E1**: Confusion matrices for all models/datasets (monolithic)
- **E2**: Distributed equivalent (using distributed engine)
- **E3**: Statistical comparison (ANOVA + Tukey HSD)
- **E4**: Scalability testing (number of nodes vs. time)
- **E5**: Fault-tolerance under node failure

See [`experiment_plan.md`](../experiments/experiment_plan.md) for full details.

---

## 📚 Documentation Index

- [Architecture Overview](./architecture/overview.md)
- [Pipeline Flow](./architecture/pipeline.md)
- [Monolithic Execution](./modules/monolithic.md)
- [Distributed Execution](./modules/distributed.md)
- [Evaluation & Visualization](./modules/evaluation.md)
- [Execution Flow Summary](./usage/execution_flow.md)

---

## 🔭 Future Extensions

The system is designed to be extensible. Planned improvements include:

- Modular and dynamic pipelines
- Custom step registration
- Container-based task execution
- Event-driven coordination
- Plugin discovery

See full details in:  
📄 [docs/architecture/future_work.md](./architecture/future_work.md)

---
