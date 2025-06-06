# Non-Container Deployment

This document describes how to deploy and run the Ensemble Learning Research System **without Docker or Docker Compose**. This approach is suitable for environments where containerization is unavailable or undesired, such as minimal Linux installations or constrained hardware (e.g., Raspberry Pi).

---

## 🧩 Structure Overview

Each node runs one or more Python modules directly. Roles are determined dynamically or passed via command-line arguments.

| Role          | Script Path                             | Description                                        |
|---------------|------------------------------------------|----------------------------------------------------|
| Experimenter  | `src/experimenter/main.py`              | Loads the YAML pipeline and initiates orchestration |
| Coordinator   | `src/distributed/coordinator.py`        | Coordinates distributed folds and assigns tasks     |
| Worker        | `src/distributed/worker.py`             | Trains base models on assigned folds               |
| Evaluator     | `src/evaluation/evaluate_results.py`    | Aggregates metrics, builds summaries               |
| Renderer      | `src/evaluation/visualize_results.py`   | Generates plots, motion canvas files, etc.         |
| Inference API | `src/inference/server.py`               | REST server for real-time predictions              |
| UI (optional) | `ui-node` (React app)                   | Web interface for job control and visualization    |
| API (optional)| `src/api/app.py`                        | Backend service for job tracking and node registry |

---

## 🔌 Communication Setup

- **LAN** , **VPN** or **Publicly available IPs** are required for distributed mode.
- `coordinator.py` listens on a socket (e.g., TCP or WebSocket).
- Workers connect to the coordinator using `--connect-to` or config.
- Shared results can be synced via:
  - NAS/NFS mount
  - Manual rsync
  - HTTP push (`PUT /upload` to API/Evaluator)

---

## 🚀 Manual Startup Sequence

### 1. Launch the Experimenter
```bash
python3 src/experimenter/main.py --pipeline ./experiment_pipeline.yml
````

### 2. Start the Coordinator

```bash
python3 src/distributed/coordinator.py --port 9000
```

### 3. Start One or More Workers

```bash
python3 src/distributed/worker.py --connect-to 192.168.1.10:9000
```

### 4. Optional Modules

* Evaluate after folds complete:

  ```bash
  python3 src/evaluation/evaluate_results.py --dataset iris
  ```

* Generate visuals:

  ```bash
  python3 src/evaluation/visualize_results.py --dataset iris
  ```

* Serve trained model:

  ```bash
  python3 src/inference/server.py --model-path ./ensembles/iris/...
  ```

* Start the API backend:

  ```bash
  python3 src/api/app.py
  ```

---

## 📁 Directory Expectations

All nodes should have access to the same structure:

```bash
.
├── src/
├── data/
│   ├── configurations/
│   └── datasets/
├── results/
├── ensembles/
├── status/
└── experiment_pipeline.yml
```

If a shared storage volume is unavailable, results must be synced manually or via the API.

---

## 🧠 Dynamic Role Detection

Some scripts (like `agent-node` or experimenter) may detect their role based on:

* Command-line arguments
* Available modules
* Network reachability
* Resources (CPU, RAM)

---

## 🔄 Fault Tolerance

* Workers retry on failure.
* Coordinator failover may be manual or scripted.
* Tasks are marked as `FAILED` and re-attempted if timeout occurs.

---

## 🔧 Configuration Notes

* Environment variables can simplify deployments:

  * `ROLE=worker`
  * `COORDINATOR_HOST=192.168.1.10`
* Logs should be redirected to files for persistent inspection.

---

## 🗂️ Related Files

* `experiment_pipeline.yml`
* `status/<dataset>/fold_<i>.json`
* `ensembles/<dataset>/...`
* `rendered/diagrams/*.svg`
* `evaluate_results.py`, `visualize_results.py`

---

This deployment approach is particularly useful for:

* Raspberry Pi clusters
* Legacy machines
* Air-gapped or low-trust environments

It retains all system functionality without relying on containerization.

