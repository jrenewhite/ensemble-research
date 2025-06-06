# Multi-Container Deployment

This document describes the container-based deployment model for the Ensemble Learning Research System, where each core module is isolated into its own Docker image. This setup supports dynamic capability detection, scalable orchestration, and execution across heterogeneous devices.

---

## 📦 Containers Overview

Each container encapsulates a logical role in the system:

| Container        | Role Description                                                                                                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `agent-node`     | Self-contained logic to determine its role (experimenter, coordinator, worker, evaluator, etc.) based on available modules. It connects to the central API or other nodes to register and synchronize. |
| `api-node`       | Central REST API to manage discovery, registration, job coordination, and status queries.                                                                                                              |
| `ui-node`        | React-based web interface to upload YAMLs, view progress, browse results, and interact with the system visually.                                                                                       |
| `evaluator-node` | Computes metrics for experiments and folds based on generated outputs.                                                                                                                                 |
| `renderer-node`  | Produces visualizations (static and animated) using matplotlib, motion canvas, or other renderers.                                                                                                     |
| `inference-node` | Exposes trained ensembles as live inference services through REST APIs.                                                                                                                                |

All containers communicate via a Docker network or a physical LAN/overlay network across devices.

---

## 🔌 Node Communication

### a. Node Discovery & Registration

* `agent-node` on startup scans its modules to detect capabilities.
* It posts its profile to `api-node` using an endpoint like `POST /register`.
* Capabilities include: `[experimenter, coordinator, worker, evaluator, renderer, inference]`

### b. Inter-Node Sync (Coordinator ↔ Workers)

* Coordinator opens a WebSocket server (or custom TCP server).
* Workers connect to it and perform handshake.
* Tasks (folds, retry, failover) are sent via messages.

### c. File Exchange (Fold results, ensembles, summaries)

* If NAS or NFS is available: agents sync to mounted volume.
* If not: data is sent over HTTP via `PUT /upload_result` to coordinator or evaluator.

---

## ⚙️ Role Determination

`agent-node` determines its role based on:

* Which containers exist in the local `docker-compose.yml`
* Which Python modules are present
* Availability of CPU, memory, and disk

Examples:

```json
{
  "node_id": "raspi-02",
  "capabilities": ["worker", "renderer"],
  "cpu_cores": 4,
  "memory_gb": 8
}
```

---

## 🧠 Fault-Tolerance & Resilience

* If an `agent-node` with coordinator role crashes, another can promote itself.
* Failed tasks due to timeout (`train.max_time_per_fold`) are reassigned.
* Nodes that become idle re-register periodically.

---

## 🌐 Web Interface

The `ui-node` interacts with `api-node` and allows users to:

* Upload `experiment_pipeline.yml`
* Track fold/task progress in real-time
* Explore plots and motion canvas animations
* Download evaluation reports or fold outputs

---

## 📡 Ports and Interfaces

| Service       | Default Port | Protocol                |
| ------------- | ------------ | ----------------------- |
| `api-node`    | 8080         | HTTP / REST             |
| `ui-node`     | 3000         | HTTP / Web UI           |
| `agent-node`  | dynamic      | WebSocket / REST client |
| `coordinator` | 9000         | WebSocket (task sync)   |
| `inference`   | 8500         | HTTP / JSON predict     |

All ports are configurable in `docker-compose.yml`.

---

## 🚀 Startup Flow Summary

1. Each node starts via Docker Compose.
2. `agent-node` scans its modules and announces itself to `api-node`.
3. `api-node` stores node profiles and exposes them to UI.
4. If `experiment_pipeline.yml` is uploaded, experimenter logic begins execution.
5. Coordinator (auto-elected) delegates folds to workers.
6. Results are synced, evaluated, and visualized.
7. Inference node may serve predictions in real-time.

---

## 🗃️ Storage

All results, plots, ensembles, and evaluation summaries are saved to a shared or local volume:

```
/results/<dataset>/<fold>/...
/status/<dataset>/<fold>.json
/ensembles/<dataset>/<ensemble_id>/...
```

NAS/NFS volumes should be mounted in Docker Compose if persistence is desired.

---

## 📎 Related Files

* `docker-compose.yml`
* `experiment_pipeline.yml`
* `status/<dataset>/fold_*.json`
* `ensembles/<dataset>/...`
* `rendered/diagrams/*.svg`

---

This architecture enables a flexible, portable, and fault-tolerant experimental system suitable for distributed research or scalable deployment across multiple devices.
