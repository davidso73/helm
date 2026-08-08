# Application Migration: VM-to-Kubernetes Architecture (EKS + Helm)

This project contains the Helm chart, Kubernetes operational manifests, and cloud configuration strategy to run the application microservices inside Amazon Elastic Kubernetes Service (AWS EKS).

---

## 1. Project Architecture

The architecture decouples the stateful database layer and cloud services from the stateless application workloads running in Kubernetes.

```text
               +-------------------------------------------------------------+
               |                    Internet / Clients                       |
               +-------------------------------------------------------------+
                                              |
                                              | HTTPS (Port 443)
                                              v
               +-------------------------------------------------------------+
               |               AWS Network Load Balancer (NLB)               |
               +-------------------------------------------------------------+
                                              |
==============================================|==============================================
KUBERNETES CLUSTER (EKS)                      v
  Namespace: app-production             +--------------+
                                        | NGINX Ingress|
                                        |  Controller  |
                                        +--------------+
                                           /        \
                            Path: /       /          \ Path: /api
                                         v            v
                                  +----------+   +----------+
                                  | Frontend |   | Backend  |
                                  | ClusterIP|   | ClusterIP|
                                  +----------+   +----------+
                                       |              |
                                       v              v
                                  +----------+   +----------+
                                  | Frontend |   | Backend  |
                                  |   Pods   |   |   Pods   |
                                  +----------+   +----------+
                                                      |
                                                      |  (Asynchronous Processing / Jobs)
                                                      v
                                                 +----------+
                                                 |  Worker  |
                                                 |   Pods   |
                                                 +----------+
======================================================|======================================
EXTERNAL MANAGED AWS INFRASTRUCTURE                   |
                                                      | (IRSA & Security Groups)
              +-----------------------+---------------+-----------------------+
              |                       |                                       |
              v                       v                                       v
   +--------------------+  +--------------------+                   +--------------------+
   |   Amazon RDS       |  |     Amazon S3      |                   |     Amazon SNS     |
   | (PostgreSQL Engine)|  |   (Object Bucket)  |                   |   (Alert Topics)   |
   +--------------------+  +--------------------+                   +--------------------+