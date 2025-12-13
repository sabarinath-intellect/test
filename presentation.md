# Comprehensive Technical Strategy: Centralized ECR & Manifest-Driven Deployment

**To:** Executive Leadership Team (CXO)  
**Date:** December 13, 2025  
**Version:** 2.0 (Detailed)  
**Prepared By:** Cloud Enablement & Governance Team  
**Key Contacts:** Sabarinath S, Selva Priya  
**Approval Authority:** Madhavan

---

## 1. Executive Summary

This document presents the technical blueprint for transforming the Purple Fabric platform's delivery pipeline. We are moving from a fragmented, multi-build approach to a **Centralized, Manifest-Driven Architecture**. This shift guarantees that the exact software artifact tested in QA is the same binary deployed to Production, governed by a strict **MAP Ticket Workflow**.

**Strategic Goals:**
1.  **Build Once, Deploy Everywhere**: Eliminate redundant builds for QA, Staging, and Prod.
2.  **Single Source of Truth**: A Centralized ECR holding immutable, scanned images.
3.  **Strict Governance**: Mandatory MAP tickets for all infrastructure changes, ensuring "Automation First".

---

## 2. The Core Problem & Solution

| Current Challenge | New Solution | Business Impact |
| :--- | :--- | :--- |
| **Redundant Builds** | **Centralized ECR**: Build Docker image *once* in Dev. Promote via retagging. | Reduces compute costs by ~70% and build times by 50%. |
| **Drift Risk** | **Manifest-Driven**: Deploy strictly from versioned JSON manifests (e.g., `v25.3.0.0`). | Eliminates "works in QA, fails in Prod" scenarios. |
| **Shadow IT** | **MAP Ticket Process**: Mandatory gate before INFRA tickets. | 100% visibility and compliance for all cloud resources. |
| **Complex Rollbacks** | **Versioned Artifacts**: Rollback is as simple as reapplying a previous manifest. | MTTR reduced from hours to minutes. |

---

## 3. Detailed Technical Architecture

### 3.1 Centralized ECR & Cross-Account Access
Instead of each environment (Dev, QA, Prod) building its own images, we utilize a **Hub-and-Spoke model**:
*   **Hub (Central Account)**: Stores all Docker images and Helm charts.
*   **Spokes (Target Accounts)**: Pull images using Cross-Account IAM Roles.

**Security & Permissions:**
*   Target accounts (e.g., Customer A, Production) are granted *Read-Only* access via specific IAM permissions:
    *   `ecr:GetDownloadUrlForLayer`
    *   `ecr:BatchGetImage`
    *   `ecr:GetAuthorizationToken`

### 3.2 Manifest-Driven Orchestration
The "Manifest" is the source of truth for every release. It acts as a bill of materials.

**Manifest Structure (`lambda-manifest-25.3.0.0.json`):**
```json
{
  "version": "25.3.0.0",
  "services": {
    "chunk-asset": {
      "image_uri": "123456789012.dkr.ecr.us-east-1.amazonaws.com/chunk-asset:qa-25.3.0.0",
      "config_path": "/PF/lambda/25.3.0.0/chunk-asset"
    },
    "embedding-gen-asset": {
      "image_uri": "123456789012.dkr.ecr.us-east-1.amazonaws.com/embedding-gen:qa-25.3.0.0"
    }
  }
}
```

**Storage Locations:**
1.  **Git (IAC Repo)**: Master branch for audit history.
2.  **AWS Parameter Store**: Active configuration for deployments (e.g., `/PF/lambda/25.3.0.0`).

---

## 4. Operational Workflows

### 4.1 Lambda Deployment (SAM-Based)
We have automated the deployment of Lambda functions using AWS SAM and CodeBuild.

**Workflow Steps:**
1.  **Source**: Developer pushes code to `source.zip` in S3.
2.  **Trigger**: Custom Lambda initiates AWS CodeBuild.
3.  **Build Phase**:
    *   Executes `pre-req.sh` to generate `application.properties` from Parameter Store.
    *   Builds Docker image and pushes to Central ECR.
4.  **Deploy Phase**:
    *   Executes `deploy.sh`.
    *   Packages SAM template.
    *   Deploys via CloudFormation with parameter overrides (e.g., `ChunkQueue`, `VPCSubnetIDS`).

### 4.2 Kubernetes Deployment (Helm-Based)
*   **Chart Management**: Helm charts are versioned and stored in the Central IAC Repository.
*   **Deployment**: Jenkins pipeline fetches the specific chart version and the corresponding image tag from the Manifest.
*   **Values Injection**: Environment-specific values (e.g., replica counts, ingress hosts) are injected at runtime from the IAC repo.

---

## 5. Governance: The MAP Ticket Process

To ensure compliance, we have introduced the **MAP Ticket Workflow**.

**Rule:** *Developers must create a MAP ticket BEFORE creating an INFRA ticket.*

### 5.1 Process Flow
1.  **Request**: Developer logs a MAP ticket in JIRA (Service Type: Create/Update/Delete).
2.  **Review**: Cloud Enablement Team validates architecture and compliance.
3.  **Automation Sync**: Team updates the `iaiautomation` environment to verify IaC.
4.  **INFRA Creation**: Only after automation sync is confirmed, an INFRA ticket is generated for Dev/QA/Prod.

### 5.2 Service Level Agreements (SLAs)
| Category | Complexity | SLA | Examples |
| :--- | :--- | :--- | :--- |
| **Category 1** | Simple | 1-3 Days | S3 Buckets, IAM Roles, Simple Lambdas |
| **Category 2** | Medium | 3-5 Days | POC Services, Simple Fee Services |
| **Category 3** | Complex | 5-10 Days | Cross-account setups, Multi-service integrations |

---

## 6. Implementation Status & Roadmap

### Phase 1: Foundation (Completed ✅)
*   Centralized ECR setup in single region.
*   IAC Repository synced to version `25.3.0.0`.
*   `iaiautomation` account deployed and validated.

### Phase 2: Integration (In Progress 🔄)
*   **Action**: Migrating Dev pipelines to push to Central ECR.
*   **Action**: Converting remaining Zip-based Lambdas to Image-based.
*   **Action**: Implementing Manifest generation logic in Jenkins.

### Phase 3: Scale (Next Steps 🚀)
*   **Goal**: Full rollout to Customer Accounts.
*   **Goal**: Automated "Promotion to QA" jobs.
*   **Goal**: Dashboard for deployment version tracking.

---

## 7. Success Metrics

| Metric | Baseline | Target |
| :--- | :--- | :--- |
| **Build Frequency** | 100% (Every Env) | **30% (Dev Only)** |
| **Deployment Time** | 45 Mins | **15 Mins** |
| **Governance Compliance** | Ad-hoc | **100% via MAP Tickets** |
| **MTTR (Recovery)** | 2 Hours | **< 30 Mins** |

---

## 8. Conclusion

The transition to a **Manifest-Driven, Centralized ECR** model is critical for scaling Purple Fabric. By coupling this technical architecture with the rigorous **MAP Ticket Governance**, we ensure that speed does not compromise stability or security.

**Immediate Request to Leadership:**
1.  Endorse the "MAP Ticket First" policy across all engineering teams.
2.  Prioritize the migration of legacy pipelines to the new Centralized ECR standard.

