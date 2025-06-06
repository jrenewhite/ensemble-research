# Ensemble Learning Distributed Platform

A modular, scalable, and distributed system for running ensemble machine learning experiments across heterogeneous nodes using dynamic roles, containerized agents, and a web interface.

---

## Features

- **Monolithic and distributed execution** with K-Fold cross-validation
- **Dynamic role assignment**: experimenter, coordinator, worker, evaluator, renderer
- **Declarative YAML pipelines** for defining experiment workflows
- **Web UI** (React-based) to monitor jobs, upload configs, and review outputs
- **Live visualization** and animation generation using D3.js and Motion Canvas
- **Task distribution** via Docker network, SSH, or shared file system (NFS/NAS)
- **Resilience & fault-tolerance** across resource-constrained nodes
- **Hardware-agnostic**: runs on x86, ARM (Raspberry Pi, Orange Pi), and macOS

---

## Architecture

Each node runs one or more containerized components, all of which coordinate via a REST API or WebSocket mesh:

| Container         | Role Description |
|------------------|------------------|
| `agent-node`      | Detects available modules and assumes a role (worker, coordinator, etc.) |
| `api-node`        | Central API for coordination, registration, and monitoring |
| `ui-node`         | Web dashboard for users |
| `evaluator-node`  | Calculates metrics for each fold |
| `renderer-node`   | Creates plots and animations |
| `inference-node`  | Hosts trained models for live prediction |

For full architecture and communication design:
- [Platform Architecture](./docs/architecture/platform_architecture.md)
- [Non-Container Deployment](./docs/architecture/non_container_deployment.md)
- [Agent Role State Machine](./docs/architecture/agent_role_state.md)
- [Pipeline Flow](./docs/architecture/pipeline.md)

---

## Experiment Pipeline

Experiments are declared using a structured YAML file. The pipeline supports:

- Sequential and parallel execution of monolithic and distributed steps
- Dynamic task reassignment
- Evaluation and visualization decoupled from training
- Inference support (real-time predictions)

See [YAML Reference](./docs/architecture/yaml_reference.md)

---

## Execution Modes

| Mode        | Description                                     |
|-------------|-------------------------------------------------|
| Monolithic  | Single-machine fold-by-fold execution           |
| Distributed | Fold-level parallelization using networked nodes|
| Hybrid      | Combined workflows for benchmarking             |
| Inference   | Use of trained ensembles for real-time scoring  |

---

## Getting Started

### Requirements

- Docker + Docker Compose (for container deployment)
- Python 3.9+ (for script-based runners)
- Shared volume or network connectivity

### Local Development

```bash
docker-compose up --build
````

### Lightweight Node (e.g. Pi)

```bash
docker-compose -f docker-compose.agent.yml up --build
```

---

## Experiments

Includes predefined scenarios:

* **E1**: Confusion matrices for all models/datasets (monolithic)
* **E2**: Distributed equivalent (multi-node benchmark)
* **E3**: Statistical comparison (ANOVA + Tukey HSD)
* **E4**: Scalability testing
* **E5**: Fault-tolerance (failures and recovery)

See full list: [`experiment_plan.md`](./docs/experiments/experiment_plan.md)

---

## Project Structure

```plaintext
src/
├── agent/           # Role negotiation and task logic
├── api/             # Flask/FastAPI service for coordination
├── ui/              # React dashboard
├── pipeline/        # YAML orchestrator
├── monolithic/      # Single-node execution runner
├── distributed/     # Coordinator and worker logic
├── evaluation/      # Metrics & statistical summaries
└── renderer/        # Plotting and animation generation
```

---

## Documentation

* [Overview](./docs/index.md)
* [Execution Flow](./docs/usage/execution_flow.md)
* [Monolithic Engine](./docs/modules/monolithic.md)
* [Distributed Engine](./docs/modules/distributed.md)
* [Evaluation & Visualization](./docs/modules/evaluation.md)
* [Inputs & Outputs](./docs/architecture/io_reference.md)
* [YAML Schema](./docs/architecture/yaml_reference.md)

---

## Contributing

1. Fork the repo
2. Use the Dockerized development environment
3. Follow clean code practices, OOP if possible
4. Add documentation in `docs/` when modifying modules
