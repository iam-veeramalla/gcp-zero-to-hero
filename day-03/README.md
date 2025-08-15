# 🔐 GCP IAM Fundamentals for Beginners 

## 🧭 What is IAM?

IAM (Identity and Access Management) in GCP is a framework that controls:
- **Who** can do **what** on **which resources**

Formula:
Who → Can do what → On which resource?

It's crucial for securely managing access in a cloud environment.

---

## 🧱 Core Concepts

| Component      | Description |
|----------------|-------------|
| **Identity**   | A user, group, domain, or service account |
| **Role**       | A collection of permissions |
| **Policy**     | A binding of identities to roles |
| **Resource**   | Any GCP object like a project, VM, bucket, etc. |

---

## 🎭 Types of Roles

| Role Type      | Description | Example |
|----------------|-------------|---------|
| **Basic Roles**| Broad, legacy roles (not recommended) | Owner, Editor, Viewer |
| **Predefined** | Fine-grained, service-specific roles | `roles/storage.admin` |
| **Custom**     | You define exact permissions | Only `compute.instances.start` |

---

## 🧬 IAM Policy Hierarchy

IAM roles can be assigned at different levels:

- **Organization**
- **Folder**
- **Project**
- **Resource**

**Note**: Permissions **inherit** from parent to child unless explicitly overridden.

---

## 🤖 Service Accounts

- **Service accounts** are non-human identities used by apps, scripts, or pipelines.
- Example: Cloud Build uses a service account to deploy apps to GKE.

---

## 🛡️ IAM Best Practices for DevOps

- ✅ Follow **Least Privilege** principle
- 🚫 Avoid **Basic Roles** (too broad)
- 🔐 Use **Service Accounts** for automation
- 🔁 Rotate keys/secrets regularly
- 📊 Audit permissions frequently

---

## 🧪 Hands-On Lab

### 🔧 Lab 1: Create Project & Assign Viewer Role

    export PROJECT_ID="devops-iam-demo-$(date +%s)"
    gcloud projects create $PROJECT_ID
    gcloud config set project $PROJECT_ID

    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="user:devops.engineer@example.com" \
      --role="roles/viewer"

---

### 🔧 Lab 2: Create a Service Account & Grant Role

    gcloud iam service-accounts create devops-bot \
      --display-name="DevOps Pipeline Bot"

    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="serviceAccount:devops-bot@$PROJECT_ID.iam.gserviceaccount.com" \
      --role="roles/cloudbuild.builds.editor"

---

## 📦 Real-World DevOps Scenario

**Use Case**:  
Your Cloud Build pipeline should deploy to GKE but NOT delete clusters.

**Solution**:
- Create a **Custom Role** with only `container.deployments.create`
- Assign that role to a **Service Account**
- Use that Service Account in your CI/CD pipeline

---

## ✅ Key Takeaways

- IAM is foundational for secure access in GCP.
- Use **Predefined or Custom Roles** — avoid basic roles.
- Service accounts are key to **DevOps automation**.
- **Always enforce least privilege and audit regularly.**

---
