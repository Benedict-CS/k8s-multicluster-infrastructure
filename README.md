# Multi-Cluster K8s Monitoring System

## Introduction

This is the documentation for a robust Hybrid Cloud Multi-Cluster Kubernetes Infrastructure. This project demonstrates a production-grade setup for managing, monitoring, and deploying applications across multiple clusters using modern cloud-native technologies.


## Components

### Kind

[kind](https://kind.sigs.k8s.io/) (Kubernetes In Docker) is a tool that enables rapid deployment and operation of Kubernetes clusters in a local environment. Leveraging Docker container technology, it simulates multiple Kubernetes nodes on a single machine, creating a Kubernetes environment for testing, development, and experimentation purposes.

### K3s

[k3s](https://k3s.io/) is a lightweight Kubernetes that retains the core functionalities of Kubernetes while removing some less utilized features, resulting in lower resource consumption and faster startup speed.


### Cilium

[Cilium](https://cilium.io/) is a software package that provides and manages network and security services for application containers running on Linux. It is designed to be scalable, efficient, and effective on containerized applications using Linux kernel features such as BPF, XDP, and IPVS.Cilium 


### Karmada

[karmada](https://karmada.io/) is a Kubernetes-based multi-cloud and multi-cluster orchestration platform. It provides central management of cloud-native workloads and applications, and enables workload and application portability across clouds and clusters.


### Prometheus

[Prometheus](https://prometheus.io/) is an open source monitoring system and time series database. It addresses many aspects of monitoring such as the generation and collection of metrics, graphing the resulting data on dashboards, and alerting on anomalies.


### Grafana

[Grafana](https://grafana.com/) is an open source visualization and analytics software. It allows you to query, visualize, alert on, and explore your metrics no matter where they are stored. In plain English, it provides you with tools to turn your time-series database (TSDB) data into beautiful graphs and visualizations.

### Thanos

[Thanos](https://thanos.io/) is a open source, highly available Prometheus setup with long term storage capabilities. It provides a global query view, high availability, data backup with historical, cheap data access as its core features in a single binary.

### HAProxy

[HAProxy](https://www.haproxy.org/) is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for web sites crawling under very high loads while needing persistence or Layer7 processing. Supporting tens of thousands of connections is clearly realistic with todays hardware.