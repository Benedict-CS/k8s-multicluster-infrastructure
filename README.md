# Hybrid Cloud Multi-Cluster Kubernetes Infrastructure

## Project Overview

This project demonstrates a robust, production-grade **Hybrid Cloud Multi-Cluster Kubernetes Infrastructure** designed for scalability, high performance, and high availability. 

The infrastructure was built on **Proxmox Virtual Environment (PVE) Type-1 virtualization**, running **Ubuntu Server 24.04 LTS** across all virtual machines (VMs). It integrates advanced cloud-native technologies to manage, monitor, and deploy applications seamlessly across multiple clusters.

### Key Features & Technologies

*   **Container Orchestration & Management:** Multi-cluster application distribution orchestrated by **Karmada**.
*   **GitOps & CI/CD Pipelines:** Automated synchronization via **Argo CD** and **Flux CD** using Helm Charts from Git repositories.
*   **Networking:** High-performance multi-cluster networking achieved through **Cilium CNI**.
*   **Observability:** A centralized monitoring stack combining **Thanos, Prometheus, and Grafana** for global metric collection and visualization.
*   **Infrastructure as Code (IaC):** Management of Kubernetes manifests and Helm Charts.
*   **Storage:** Configured Kubernetes **Persistent Volume Claims (PVCs)** for persistent storage management.

---

## System Architecture

### Multi-cluster Monitoring Architecture
![Multi-cluster Monitoring Architecture](https://hackmd.io/_uploads/H1vXiBelGg.jpg)
*Figure 1: Kubernetes multi-cluster architecture*

### Multi-cluster GitOps Pipeline
![Multi-cluster GitOps Pipeline](https://hackmd.io/_uploads/B1XjsSxezl.jpg)
*Figure 2: Multi-cluster GitOps deployment pipeline*

---

## Components Detail

### Kind & K3s
Utilized for rapid deployment and lightweight Kubernetes environments, perfect for testing and development of cluster nodes.

### Cilium
Manages network and security services using Linux kernel features such as BPF and XDP for maximum efficiency.

### Karmada
Provides central management and orchestration for cloud-native workloads across multi-cloud and multi-cluster environments.

### Monitoring Stack (Thanos + Prometheus + Grafana)
*   **Prometheus**: Metrics collection and alerting.
*   **Thanos**: Provides a global query view and long-term storage capabilities for highly available Prometheus setups.
*   **Grafana**: Advanced visualization and analytics dashboard.

---

## Laboratory Guide

This repository contains a comprehensive 15-lab series covering the entire setup process:

1.  **Lab 01**: Cilium Setup
2.  **Lab 02**: Cilium Gateway API
3.  **Lab 03**: Simple App Deployment
4.  **Lab 04**: Hubble Observability
5.  **Lab 05**: Prometheus and Grafana
6.  **Lab 06**: Cluster Mesh
7.  **Lab 07**: Multi-cluster Setup
8.  **Lab 08**: Karmada Setup
9.  **Lab 09**: Simple App Propagating
10. **Lab 10**: HAProxy Setup
11. **Lab 11**: Multi-cluster Failover
12. **Lab 12**: Network Policy
13. **Lab 13**: Multi-cluster Monitoring
14. **Lab 14**: Alert Manager
15. **Lab 15**: GitOps (ArgoCD/FluxCD)

---

## Related Repositories
*   [k8s-multicluster-gitops](https://github.com/Benedict-CS/k8s-multicluster-gitops): GitOps Manifests and Helm Charts used for automated deployment in this infrastructure.
