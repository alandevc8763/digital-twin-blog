---
title: "Sre_Infra"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Hubs", "Sre Infra.Md"]
draft: false
---

# 🛠️ SRE & Infrastructure Hub

Welcome to the SRE Knowledge Map. Here, we dive deep into the Linux kernel, networking stacks, and the art of building stable, high-performance systems.

## 🚀 Performance & Kernel
*Focus: Squeezing every drop of performance from the hardware.*
- **CPU & Memory**: [[articles/performance/cpu-numa|CPU Topology & NUMA Penalties]] $\rightarrow$ [[articles/performance/cfs-scheduler|CFS Scheduler Deep Dive]]
- **I/O Path**: [[articles/performance/vfs-tracing|VFS & Page Cache Tracing]] $\rightarrow$ [[articles/performance/psi-metrics|PSI Pressure Metrics]]
- **Network Stack**: [[articles/tcp-stack-deep-dive|TCP Stack Internals]] $\rightarrow$ [[articles/network-tuning-deep-dive|Network Tuning Guide]]
- **Observability**: [[articles/ebpf-modern-observability|eBPF Modern Observability]]

## 📦 Container & Orchestration
*Focus: The plumbing of modern cloud-native environments.*
- **Isolation**: [[articles/container-isolation-internals|Namespace & Cgroups]] $\rightarrow$ [[articles/container-runtime-deep-dive|Runtime Architecture]]
- **K8s Internals**: [[articles/k8s-storage-plumbing|Storage Plumbing]] $\rightarrow$ [[articles/k8s-networking-plumbing|Networking Plumbing]] $\rightarrow$ [[articles/k8s-pod-lifecycle|Pod Lifecycle]]
- **Advanced K8s**: [[articles/k8s-resource-limits|Resource Limits Truth]] $\rightarrow$ [[articles/k8s-gateway-api-evolution|Gateway API Evolution]] $\rightarrow$ [[articles/gpu-llm-k8s-scheduling|GPU Scheduling]]

## ☁️ Cloud & Data Engineering
*Focus: Scaling and managing distributed systems.*
- **Messaging**: [[cloud_infra/aws-msk|AWS MSK]] $\rightarrow$ [[cloud_infra/kafka/design-philosophy|Kafka Design Philosophy]] $\rightarrow$ [[cloud_infra/kafka/ops-guide|Kafka Ops Guide]]
- **Identity**: [[cloud_infra/iam-cli|IAM & CLI Mastery]]
- **Databases**: [[db_eng/mongodb|MongoDB Internals]]

---
**Next Step $\rightarrow$** If you are building a local lab to test these, visit the [[hubs/homelab|HomeLab Hub]].