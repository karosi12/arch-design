## Innovate Inc. Cloud Infrastructure Architecture (AWS).

## Overview

This document outlines a robust, scalable, secure, and cost-effective cloud architecture for Innovate Inc, leveraging AWS Elastic Kubernetes Service (EKS), Amazon RDS PostgreSQL, and CI/CD best practices.


-  ## Cloud Environment Structure

Recommended Account Structure: I recommend a multi-account setup using AWS Organizations, aligned with best practices:

| Account          | Purpose                                    |
|------------------|--------------------------------------------|
| Management       | Root billing and organization management.  |
| Development      | Hosts development and staging environments.|
| Production       | Hosts production workloads with stricter controls.                    |
| Security/Logging | Centralized security, monitoring, logging  |


### Justification:

Isolation: Minimizes conflicts between environments.

Security: Separate roles/policies that is everything that has to do with permission and protection of the infrastruture (IAM, OIDC).

Billing: Clear cost tracking and budgeting (Billings).

Scalability: Easily extendable to future teams or services.

- ## Network Design

VPC Architecture - per environment

- Regions: Use 1 primary AWS Region(e.g eu-central-1) to start, plan for multi-region failover later.
- VPC: One VPC per environment.
- Subnets:
    - Public Subnets: For ALB, NAT gateways.
    - Private Subnets: For EKS nodes, RDS.
    - Isolated Subnets: For database (no internet access).

- NAT Gateway: 1 per AZ(up to 3AZs) to enable private subnets outbound access
- Route Tables and Security Groups: Least priviledges rules with flow logs enabled on the VPC level. The Route and Security group help secure how traffic flow through the infrastructure. While The flow logs capture information about the IP going to and from the network interfaces. It help to view log how traffic flow throught the VPC.

#### Security Measures
- VPC Flow logs to monitor traffic.
- Security Groups per component (e.g: ALB SG, EKS Worker SG).
- NACLs for subnet-level hardening.
- WAF + Shield for L7 protection.
- IAM roles with fine-grained leadt-prividesge access.

- ## Compute Platform (EKS)

### Kubernets Setup
- #### Amazon EKS (manged control plane).
- #### Node Groups:
    - ##### Managed Node Groups(MNG) for general workloads.
    - ##### Karpenter: for dynamic autoscaling with Spot + On-demand mix (cost optimization).
    - Separate node groups for
        - App Workloads(CPU-intensive).
        - Database proxy/cache (memory-optimized).
        - CI/CD agents(tainted nodes).

### Scaling & Resources

- #### Horizontal Pod Autoscaler(HPA): for application scaling
- #### Cluster Autoscaler/Karpenter: to scale node groups.
- #### Resource Request & Limits:  defined for all pods.
- #### Priority Classes: to ensure critical workloads are scheduled.

### Scaling & Resources

- #### Docker: for building images(using multi=stage builds).
- #### Amazon ECR:  for secure container image storage.
- #### CI/CD:
    - Github Actions.
    - docker build --> ecr push --> kubectl apply
- ####  Deployment:
    - Helm for templating and versioning.
    - ArgoCD for GitOps-based deployment.

- ## Database Layer
### Recommended service

- #### Amazon RDS for PostgreSQL(managed).
    - Enables backups, high availability (Multi-AZ), automated patching.
    - Option to use Aurora PostgreSQL in future for better scalability.
- ### High Availablity& Data Recovery
    - Multi-AZ deployment(automatic failover).
    - Automated Backups (retained for 7 - 30 days).
    - Manual Snapshots for specific milestones/releases.
- ### Security:
    - Encrypted at rest(KMS) and in transit(SSL).
    - Use IAM authentication or rotate credentials via Secrets Manager, and DB-level user management.
    - Private subnet, no direct public access.

- ## CI/CD with DevOps Tooling
    - Github/Gitlab for source control and CI/CD.
    - CI/CD pipeline:
        - Run tests(linting, unit tests).
        - Build Docker images.
        - Deploy to staging -> run integration tests -> promote to production
    - Argo CD for GitOps sync
    - Secrets Management: AWS Secrets Manager or HashiCorp Valut or Sealed Secret for K8S.