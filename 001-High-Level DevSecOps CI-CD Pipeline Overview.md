# 🔧 High-Level DevSecOps CI-CD Pipeline Overview

Your pipeline is broken down into **5 core stages**:

```
📁 Plan → 🏗️ Build → 🧪 Test → 🚀 Deploy → 📊 Monitor
```

---

### 🧭 **1. PLAN** (Infrastructure & Configuration)

| Tool             | Purpose                                                               |
| ---------------- | --------------------------------------------------------------------- |
| 🌍 **Terraform** | Provision AWS EC2 instance & basic infra (EBS, security groups, etc.) |
| ⚙️ **Ansible**   | Configure software and tools (Docker, Jenkins, K3s, SonarQube, etc.)  |

> 📝 *All provisioning and setup is automated and idempotent.*

---

### 🏗️ **2. BUILD** (Source, Compile, and Package Code)

| Tool                | Purpose                                              |
| ------------------- | ---------------------------------------------------- |
| 🔁 **Git & GitHub** | Host and manage source code with version control     |
| 🧩 **Jenkins**      | Orchestrates CI/CD pipeline triggers and jobs        |
| ☕ **Maven**         | Builds and packages Java application (e.g., JAR/WAR) |

> 📝 *Build is triggered automatically on GitHub commits or pull requests.*

---

### 🧪 **3. TEST** (Quality, Security, and Compliance)

| Tool             | Purpose                                                                      |
| ---------------- | ---------------------------------------------------------------------------- |
| 🔍 **SonarQube** | Performs static code analysis and detects code smells, bugs, vulnerabilities |
| 🛡️ **Trivy**    | Scans Docker images for vulnerabilities before deployment                    |

> 📝 *Jenkins runs quality gates and security scans after successful builds.*

---

### 🚀 **4. DEPLOY** (Containerize & Orchestrate)

| Tool                    | Purpose                                                                           |
| ----------------------- | --------------------------------------------------------------------------------- |
| 🐳 **Docker**           | Packages the application into containers                                          |
| ☸️ **Kubernetes (K3s)** | Lightweight single-node K8s for container orchestration (ideal for t2.medium EC2) |

> 📝 *Once scans pass, Docker images are deployed to K3s via Jenkins.*

---

### 📡 **5. MONITOR** (Visibility, Alerting, and Metrics)

| Tool              | Purpose                                                            |
| ----------------- | ------------------------------------------------------------------ |
| 📈 **Prometheus** | Collects metrics from Jenkins, K3s, Node, and apps                 |
| 📊 **Grafana**    | Visualizes Prometheus data for observability dashboards and alerts |

> 📝 *You’ll get real-time health and performance metrics of your pipeline and deployed services.*

---

## 🖥️ Single EC2 Instance Setup Overview

| Specification | Value                                                                      |
| ------------- | -------------------------------------------------------------------------- |
| OS            | Ubuntu 24.04 LTS                                                           |
| EC2 Type      | t2.medium (2 vCPUs, 4 GB RAM)                                              |
| Storage       | 50 GB EBS                                                                  |
| Hosting       | Everything runs on this single instance (via lightweight, optimized setup) |

> 🧠 *We’ll use K3s instead of full Kubernetes for resource efficiency. SonarQube and Jenkins will be configured with optimal Java heap size to prevent memory bottlenecks.*

---
