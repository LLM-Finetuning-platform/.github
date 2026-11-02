## Hi there 👋
<div align="center">

<img src="https://github.com/LLM-Finetuning-platform.png" width="96" height="96" style="border-radius: 50%; margin-bottom: 12px;" alt="LLM Finetuning Platform" />

# LLM Finetuning Platform (LFP)

**Production-grade, unbundled distributed ML orchestration for Kubernetes.**

<p>
  <a href="https://github.com/LLM-Finetuning-platform"><img src="https://img.shields.io/badge/Control_Plane-Go_%7C_gRPC-00ADD8?style=flat-square&logo=go&logoColor=00ADD8&labelColor=161b22" alt="Control Plane" /></a>
  <a href="https://github.com/LLM-Finetuning-platform"><img src="https://img.shields.io/badge/Orchestration-Temporal-ffffff?style=flat-square&logo=temporal&logoColor=white&labelColor=161b22" alt="Temporal" /></a>
  <a href="https://github.com/LLM-Finetuning-platform"><img src="https://img.shields.io/badge/Compute-KubeRay_%7C_PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=EE4C2C&labelColor=161b22" alt="Compute" /></a>
  <a href="https://github.com/LLM-Finetuning-platform"><img src="https://img.shields.io/badge/Infrastructure-Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=326CE5&labelColor=161b22" alt="Kubernetes" /></a>
  <a href="https://github.com/sponsors/LLM-Finetuning-platform"><img src="https://img.shields.io/badge/Sponsor-%E2%9D%A4-ea4aaa?style=flat-square&logo=github&logoColor=white&labelColor=161b22" alt="Sponsor" /></a>
</p>

</div>

---

### Overview

Most fine-tuning workflows rely on fragile single-node scripts that fail under cluster preemption and multi-GPU memory pressure. **LFP** provides a modular, Kubernetes-native platform that unifies API ingress and project orchestration in Go, while isolating authentication, data preparation, and distributed training into dedicated, resilient microservices communicating over gRPC.

---

### System Architecture

```text
                           [ External Traffic / Web / CLI ]
                                          │
                                          ▼ (HTTP/REST)
                      ┌───────────────────────────────────────────────┐
                      │            gateway-project-service            │ (Go)
                      │  Ingress • Project Metadata • Cluster Control │
                      └───────┬───────────────┬───────────────┬───────┘
                              │               │               │
            ┌─────────────────┘               │               └─────────────────┐
            │ gRPC                            ▼ (SQL)                           │ (Workflow Dispatch)
            ▼                          ┌──────────────┐                         ▼
 ┌─────────────────────┐               │  PostgreSQL  │              ┌─────────────────────┐
 │    auth-service     │               │  (DB State)  │              │  Temporal Cluster   │
 │ (JWT, RBAC, Tokens) │               └──────────────┘              │ (State & Retries)   │
 └─────────────────────┘                                             └──────────┬──────────┘
                                                                                │
                                           ┌────────────────────────────────────┴────────────────────────────────────┐
                                           │ Activity Dispatch                                                       │ Activity Dispatch
                                           ▼                                                                         ▼
                              ┌─────────────────────────┐                                               ┌─────────────────────────┐
                              │    data-prep-service    │                                               │   KubeRay Cluster Pods  │
                              │   (K8s Worker Pods)     │                                               │ (Containerized Training)│
                              │ Tokenization, Chunking, │                                               │  ┌───────────────────┐  │
                              │  Validation & S3/PVC    │                                               │  │llmfinetune-service│  │
                              └─────────────────────────┘                                               │  │ Ray Actors + FSDP │  │
                                                                                                        │  └───────────────────┘  │
                                                                                                        └─────────────────────────┘
```
#### Communication & Execution Flow

* **Edge & Project Ingress:** The unified Go `gateway-project-service` handles all incoming client HTTP requests, validates tokens and RBAC rules via `auth-service` over gRPC, and reads/writes project runs and workspace state directly to PostgreSQL.
* **Cluster & Workflow Dispatch:** When a training run is scheduled, `gateway-project-service` initiates a durable workflow inside the **Temporal** cluster.
* **Data Preparation Pipeline:** Temporal dispatches activity tasks to `data-prep-service` Kubernetes pods to ingest, clean, tokenize, and shard datasets onto shared PVCs or object storage.
* **Distributed Training:** Temporal coordinates with the **KubeRay** cluster to provision worker pods running the containerized `llmfinetune-service`, executing multi-GPU fine-tuning routines (PyTorch FSDP, LoRA, QLoRA).

---

### Core Repositories & Schemas

| Repository / Registry | Stack | Role |
| :--- | :--- | :--- |
| **[`go-auth-api`](https://github.com/LLM-Finetuning-platform/go-auth-api)** | `Go` `gRPC` `Postgres` | Authentication service managing API tokens, JWT issuance, session states, and RBAC verification. |
| **[`llm-finetuning-service`](https://github.com/LLM-Finetuning-platform/llm-finetuning-service)** | `Python` `Ray` `Temporal` `PyTorch` | Containerized distributed fine-tuning worker running PyTorch FSDP and QLoRA on KubeRay pods. |
| **[`shared-workflows`](https://github.com/LLM-Finetuning-platform/shared-workflows)** | `GitHub Actions` | Reusable CI/CD pipelines, Docker container build matrices, linting checks, and testing workflows. |
| **[`buf.build/llmfp`](https://buf.build/llmfp)** | `Buf Schema Registry` `gRPC` | Centralized Protobuf schema registry hosting type-safe gRPC contracts and generated code bindings. |

---


### 🎯 Engineering Focus & Allocation

```text
[ KubeRay Distributed Engine & PyTorch FSDP  ]  ████████████████░░░░░░░░  45%
[ Temporal Resilient Workflows & Data Prep   ]  ██████████░░░░░░░░░░░░░░  30%
[ Go Gateway, Project Store, Auth & gRPC     ]  ████████░░░░░░░░░░░░░░░░  25%
```
---

### Maintainers

<table>
  <tr>
    <td align="center" width="50%">
      <a href="https://github.com/eswarashish">
        <img src="https://github.com/eswarashish.png" width="85" height="75" style="border-radius: 50%;" alt="Ashish Kusumanchi" />
      </a>
      <br /><br />
      <b>Eswar Ashish Kusumanchi</b>
      <br />
      <sub>Creator, Sole Builder & Core Maintainer</sub>
      <br /><br />
      <a href="https://www.youtube.com/@ashhhh-h2m">
        <img src="https://img.shields.io/badge/Build%20Logs-YouTube%20Streams-FF0000?style=flat-square&logo=youtube&logoColor=white" alt="YouTube Stream Logs" />
      </a>
      <img src="https://img.shields.io/badge/Focus-End--to--End%20Implementation-326CE5?style=flat-square" alt="Focus" />
      <p align="left"><sub>Designed and wrote 100% of the platform codebase from scratch—streamed entirely live on YouTube. Engineered the unified Go gateway, internal gRPC contracts, Temporal workflow state machines, and the Ray/PyTorch distributed training execution engine on Kubernetes.</sub></p>
    </td>
    <td align="center" width="50%">
      <a href="https://github.com/Nithin029">
        <img src="https://github.com/Nithin029.png" width="85" height="75" style="border-radius: 50%;" alt="Contributor" />
      </a>
      <br /><br />
      <b>Oruganti Nithin Sharma</b>
      <br />
      <sub>Technical & Design Assistancer</sub>
      <br /><br />
      <img src="https://img.shields.io/badge/Focus-System%20Design%20Review-00ADD8?style=flat-square" alt="Focus" />
      <p align="left"><sub>Acts as an architectural sounding board. Consults on high-level system design questions, edge-case validation, and distributed architecture trade-offs.</sub></p>
    </td>
  </tr>
</table>

---

### Support Our work

 
  *Open-source distributed MLOps platform for scalable PyTorch model training, orchestration, and inference.*
  * [![Sponsor Org](https://img.shields.io/badge/Sponsor-%E2%9D%A4-ea4aaa?style=flat-square&logo=github&logoColor=white&labelColor=161b22)](https://github.com/sponsors/LLM-Finetuning-platform) - **Direct project sponsorship**
  * [![One-time Tier](https://img.shields.io/badge/Sponsor-%E2%9D%A4-ea4aaa?style=flat-square&logo=github&logoColor=white&labelColor=161b22)](https://github.com/sponsors/LLM-Finetuning-platform?frequency=one-time) - **Buy compute hours**
  * [![Monthly Tier](https://img.shields.io/badge/Sponsor-%E2%9D%A4-ea4aaa?style=flat-square&logo=github&logoColor=white&labelColor=161b22)](https://github.com/sponsors/LLM-Finetuning-platform?frequency=recurring) - **Sustain ongoing development**

---
---
