# ✅ High-Level DevSecOps Pipeline Overview

```
+-------------------------------------------------------------------------------------+
|                               DevSecOps CI/CD Pipeline                             |
+----------------------+--------------------+--------------------+--------------------+
|       🧠 PLAN        |      🔧 BUILD      |     🔍 TEST        |     🚀 DEPLOY      |
+----------------------+--------------------+--------------------+--------------------+
| Terraform            | Jenkins            | SonarQube          | Jenkins            |
| (Infra as Code)      | Maven              | Trivy (Docker Scan)| Docker             |
| AWS EC2, Security    | GitHub Webhook     | Unit + Sec tests   | Kubernetes (K3s)   |
| Groups, IAM          | Git integration    | Quality Gate       | Helm (optional)    |
|                      |                    |                    |                    |
+----------------------+--------------------+--------------------+--------------------+
|                          📈 MONITOR (Post-Deployment)                             |
+-------------------------------------------------------------------------------------+
| Prometheus (Metrics) + Grafana (Dashboards)                                        |
| Monitors Jenkins, K8s, Node Exporter, App health                                  |
+-------------------------------------------------------------------------------------+
```

---

### 🔁 Pipeline Flow Diagram

```plaintext
                ┌─────────────┐
                │   GitHub    │◄────────────┐
                └─────┬───────┘             │
                      │ Push (Webhooks)     │
                      ▼                     │
                 ┌────────────┐             │
       ┌───────► │  Jenkins   │ ─────────┐  │
       │         └────────────┘          │  │
       │          🧠 PLAN (Terraform)    │  │
       │          🔧 BUILD (Maven)       │  │
       │          🔍 TEST (SonarQube,    │  │
       │                    Trivy)       │  │
       │          🚀 DEPLOY (Docker +    │  │
       │                       K3s)      │  │
       │                                 ▼  ▼
┌──────┴─────┐                     ┌────────────┐
│   Ansible  │ ◄───────────────►  │   EC2 Host │
│ (Provision)│                    └────┬───────┘
└────────────┘                         │
                                       ▼
                         ┌───────────────────────────┐
                         │ Prometheus + Grafana      │
                         │ (Monitor Jenkins, K8s, App)│
                         └───────────────────────────┘
```

---

### 🔐 Security & Best Practices Embedded Throughout:

| Stage   | Security Aspects                                  |
| ------- | ------------------------------------------------- |
| PLAN    | Least privilege IAM roles, Terraform state safety |
| BUILD   | GitHub branch protection, Jenkins agent isolation |
| TEST    | Static (SAST) & Dependency (SCA) scanning         |
| DEPLOY  | Trivy vulnerability gates, signed Docker images   |
| MONITOR | Audit logs, Prometheus alerting                   |

---

### ✅ Summary of Tool Usage per Role

| Tool           | Role in Pipeline                                   |
| -------------- | -------------------------------------------------- |
| **Terraform**  | Provision AWS infra (EC2, IAM, etc.)               |
| **Ansible**    | Bootstrap EC2 (install Jenkins, K3s, Docker, etc.) |
| **GitHub**     | Source control, triggers pipeline                  |
| **Jenkins**    | Orchestrator of the pipeline                       |
| **Maven**      | Builds Java artifacts                              |
| **SonarQube**  | Static code analysis (SAST)                        |
| **Trivy**      | Image vulnerability scanning                       |
| **Docker**     | Containerize app and Jenkins agents                |
| **K3s**        | Lightweight Kubernetes for deploying app           |
| **Prometheus** | Metrics and monitoring                             |
| **Grafana**    | Dashboards and alerts                              |

---
