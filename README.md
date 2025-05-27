# Ensemble Learning Distributed Platform

A modular, scalable, and distributed system for running ensemble machine learning experiments across heterogeneous nodes using dynamic roles, containerized agents, and a web interface.

---

## Features

- **Monolithic and distributed training** (fold-by-fold ensemble learning)
- **Dynamic role assignment**: experimenter, coordinator, worker, evaluator
- **Declarative YAML pipelines**
- **Web GUI** to monitor experiments and submit configurations
- **Task distribution** via Docker, SSH, or shared folders
- **Visualization engine** using D3.js and optional Motion Canvas animations
- **Compatible with heterogeneous hardware** (x86, ARM, etc.)

---

## Architecture

Each node runs a set of containerized services:

| Container      | Description                             |
|----------------|-----------------------------------------|
| `agent-node`   | Executes tasks and negotiates role       |
| `api-node`     | Central API and task dispatcher          |
| `ui-node`      | React web interface                      |
| `evaluator-node` | Optional, generates visuals separately |

All roles can run on any node. The system is designed to be hardware-agnostic and auto-configuring.

See [Platform Architecture](./docs/architecture/platform_architecture.puml)  
See [Role State Machine](./docs/architecture/agent_role_state.puml)

---

## Getting Started

### Requirements

- Docker + Docker Compose
- Python 3.9+ (for local dev or scripts)
- Shared volume or network access between nodes

### Quick Start (All roles in one node)

```bash
docker-compose up --build
````

### Lightweight Agent (e.g., Raspberry Pi)

```bash
docker-compose -f docker-compose.agent.yml up --build
```

---

## Experiments

Includes support for executing predefined experiment scenarios:

* **E1**: Monolithic confusion matrix evaluation
* **E2**: Distributed equivalent
* **E3**: ANOVA + Tukey post-hoc comparison
* **E4**: Scalability benchmark
* **E5**: Fault-tolerance under partial failure

See [`experiment_plan.md`](./docs/experiments/experiment_plan.md)

---

## Dynamic Roles

Each `agent-node` selects a role at runtime based on:

* Boot mode (`BOOT_MODE=auto|experimenter|worker|...`)
* Discovery of existing experimenters or coordinators
* Resource availability

See [Dynamic Role Assignment](./docs/architecture/roles.md)

---

## Visualization

* In-browser charts powered by **D3.js**
* Offline animated videos generated using **Motion Canvas**

---

## Project Structure

```plaintext
src/
  ├── pipeline/         # Core pipeline runner and step handlers
  ├── agent/            # Role negotiation and executor logic
  ├── api/              # REST API backend
  ├── ui/               # React-based web frontend
  └── evaluation/       # Evaluation & visualization logic
```

---

## Contributing

1. Fork the repo
2. Use the Dockerized dev environment
3. Write clean, modular code with comments in English
4. Document your work under `docs/`


