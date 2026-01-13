# Kubernetes Laboratory: Sysadmin's Log

## Overview
This repository serves as a practical, hands-on field guide for Systems Administrators migrating to or learning Kubernetes. It is designed as a "Lab Notebook"—clean, precise, and focused on reproducible results using **Minikube**.

## Prerequisite Knowledge
*   Basic Linux CLI proficiency.
*   Understanding of standard networking (IPs, Ports, DNS).
*   Basic understanding of Containers (Docker/Podman).

## Module Index

### [00. Environment Setup](./00-setup/README.md)
*   **Objective:** Establish a local Kubernetes control plane.
*   **Tools:** Minikube, kubectl.
*   **Platform:** Linux / macOS.

### [01. The Basics: Pods & Deployments](./01-basics/README.md)
*   **Objective:** Deploy and manage stateless applications.
*   **Concepts:** Pods, ReplicaSets, Deployments, Services (ClusterIP, NodePort).

### [02. Configuration Management](./02-configuration/README.md)
*   **Objective:** Decouple configuration from container images.
*   **Concepts:** ConfigMaps, Secrets, Environment Variables.

### [03. Storage & State](./03-storage/README.md)
*   **Objective:** Persisting data beyond the lifecycle of a Pod.
*   **Concepts:** PersistentVolumes (PV), PersistentVolumeClaims (PVC), StorageClasses.

### [04. Networking & Ingress](./04-networking/README.md)
*   **Objective:** Exposing services to the external world efficiently.
*   **Concepts:** Ingress, Ingress Controllers.

---
*Created: 2026-01-13*
*Status: Active*
