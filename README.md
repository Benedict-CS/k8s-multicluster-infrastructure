# Hybrid Cloud Multi-Cluster Kubernetes Infrastructure

## Project Overview

This project demonstrates a robust, production-grade **Hybrid Cloud Multi-Cluster Kubernetes Infrastructure** designed for scalability, high performance, and high availability. It integrates advanced cloud-native technologies to manage, monitor, and deploy applications seamlessly across multiple clusters, providing a unified control plane and global observability.

### Key Features & Technologies

*   **Container Orchestration & Management:** Multi-cluster application distribution orchestrated by **Karmada**.
*   **GitOps & CI/CD Pipelines:** Automated synchronization via **Argo CD** and **Flux CD** using Helm Charts from Git repositories.
*   **Networking:** High-performance multi-cluster networking achieved through **Cilium CNI**.
*   **Observability:** A centralized monitoring stack combining **Thanos, Prometheus, and Grafana** for global metric collection and visualization.
*   **Infrastructure as Code (IaC):** Management of Kubernetes manifests and Helm Charts.
*   **Storage:** Configured Kubernetes **Persistent Volume Claims (PVCs)** for persistent storage management.

---

## System Architecture

### Multi-cluster Infrastructure Architecture
![Multi-cluster Infrastructure Architecture](./labs/assets/architecture-overview.jpg)
*Figure 1: Kubernetes multi-cluster architecture overview*

### Multi-cluster GitOps Pipeline
![Multi-cluster GitOps Pipeline](https://hackmd.io/_uploads/B1XjsSxezl.jpg)
*Figure 2: Multi-cluster GitOps deployment pipeline*

---

## Components Detail

### Kind
[kind](https://kind.sigs.k8s.io/) (Kubernetes In Docker) is a tool that enables rapid deployment and operation of Kubernetes clusters in a local environment. Leveraging Docker container technology, it simulates multiple Kubernetes nodes on a single machine.

### K3s
[k3s](https://k3s.io/) is a lightweight Kubernetes that retains core functionalities while removing less utilized features, resulting in lower resource consumption and faster startup speed.

### Cilium
[Cilium](https://cilium.io/) provides and manages network and security services for application containers using Linux kernel features such as BPF, XDP, and IPVS for maximum efficiency.

### Karmada
[karmada](https://karmada.io/) is a Kubernetes-based multi-cloud and multi-cluster orchestration platform. It provides central management of cloud-native workloads and applications across clouds and clusters.

### Monitoring Stack (Thanos + Prometheus + Grafana)
*   **Prometheus**: An open source monitoring system and time series database for generating and collecting metrics.
*   **Thanos**: Highly available Prometheus setup with long-term storage capabilities and a global query view.
*   **Grafana**: Visualization and analytics software for turning time-series data into beautiful graphs and dashboards.

### HAProxy
[HAProxy](https://www.haproxy.org/) is a fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications.

---

## Laboratory Guide

This repository contains a comprehensive 15-lab series covering the entire setup process:

1.  **[Lab 01: Cilium Setup](./labs/lab01-cilium-setup)**
2.  **[Lab 02: Cilium Gateway API](./labs/lab02-cilium-gateway-api)**
3.  **[Lab 03: Simple App Deployment](./labs/lab03-simple-app-deployment)**
4.  **[Lab 04: Hubble Observability](./labs/lab04-hubble-observability)**
5.  **[Lab 05: Prometheus and Grafana](./labs/lab05-prometheus-and-grafana)**
6.  **[Lab 06: Cluster Mesh](./labs/lab06-cluster-mesh)**
7.  **[Lab 07: Multi-cluster Setup](./labs/lab07-multi-cluster-setup)**
8.  **[Lab 08: Karmada Setup](./labs/lab08-karmada-setup)**
9.  **[Lab 09: Simple App Propagating](./labs/lab09-simple-app-propagating)**
10. **[Lab 10: HAProxy Setup](./labs/lab10-haproxy-setup)**
11. **[Lab 11: Multi-cluster Failover](./labs/lab11-multi-cluster-failover)**
12. **[Lab 12: Network Policy](./labs/lab12-network-policy)**
13. **[Lab 13: Multi-cluster Monitoring](./labs/lab13-multi-cluster-monitoring)**
14. **[Lab 14: Alert Manager](./labs/lab14-alert-manager)**
15. **[Lab 15: GitOps (ArgoCD/FluxCD)](./labs/lab15-gitops)**

---

## Related Repositories
*   [k8s-multicluster-gitops](https://github.com/Benedict-CS/k8s-multicluster-gitops): GitOps Manifests and Helm Charts used for automated deployment in this infrastructure.
