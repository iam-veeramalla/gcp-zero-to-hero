# GCP Zero to Hero

## Day-1: Introduction to Cloud and GCP

- What is Cloud Computing? Cloud vs Traditional IT
- Why GCP? GCP vs AWS vs Azure (with honest comparison)
- Setup GCP free account 
- Understanding GCP Console and Cloud Shell

---

## Day-2: GCP Project Setup and Billing

- Projects, billing accounts, and quotas
- Creating and linking projects to billing
- Budget alerts and cost visibility
- Enabling essential APIs (Compute, Storage, etc.)

---

## Day-3: IAM – Roles, Permissions, and Best Practices

- GCP IAM basics – who can access what?
- Roles: Basic, Predefined, Custom
- Use-case: Give limited access to a junior teammate
- Principle of least privilege and real-world security tips

---

## Day-4: Compute Engine – VMs, SSH, Types of Compute, Region, Zone and much more

- Launch VMs, connect via SSH
- Install packages via startup scripts
- Create and use custom images
- Enable firewall rules for external access

---

## Day-5: Cloud Storage – Buckets, Files, Datastore, Pub/Sub.

- Google Cloud Storage services
- Comaparison and difference between the services
- Focus on GCS: Create buckets, upload/download files
- Object versioning, lifecycle policies (coldline, nearline)
- Static website hosting with GCS
- Bucket permissions (fine-grained vs uniform)

Demo: GCS with Compute Engine VM
- Create a GCS Bucket and upload/download files.
- Use a Service Account to give secure, keyless access.
- Launch a Compute Engine VM with attached IAM role.
- Access GCS from inside the VM (read/write files).
- Learn project cleanup and what happens when a project is deleted.

---

## Day-6: VPC Networking – Subnets and Firewalls

- Default vs Custom VPC
- IP ranges, subnet planning (CIDRs)
- Configure firewall rules (SSH, HTTP)
- Network tags and internal communication

---

## Day-7: Load Balancing + Managed Instance Groups

- Create MIGs for autoscaling web servers
- Setup HTTP(S) Load Balancer with health checks
- Real-world HA example: App with 3 zones
- Add logging to monitor backend status

---

## Day-8: Private App on Custom VPC + Regional External HTTP(S) LB + Custom Domain (GoDaddy + Cloud DNS)

- What you’ll build (end-to-end)
  - A custom VPC with **public** and **private** subnets, plus a **proxy-only** subnet
  - A simple Python **Flask** app running on private VMs (no external IPs), managed by a **Managed Instance Group**
  - A **Regional External HTTP(S) Load Balancer** in the public tier
  - A public **Cloud DNS** zone mapped to a **GoDaddy** domain so the app is reachable via a friendly name
  - Outbound internet for private VMs via **Cloud NAT**
    
---

## Day-9: Monitoring and Logging

- Cloud Monitoring – create dashboard for VM metrics
- Cloud Logging – view logs, set alerts
- Uptime checks and alerting policies
- Use-case: Send Slack/Email alert on high CPU usage

---

## Day-10: Cloud SQL (PostgreSQL) – Setup and Access

- Create a managed PostgreSQL DB
- Secure access from VM using private IP
- Manage DB users and backups
- Import/export database dumps

---

## Day-11: Secrets Manager – Storing API Keys Safely

- Why not to hardcode passwords or tokens
- Store and access secrets securely
- IAM-based access control for secrets
- Rotate and audit access logs

---

## Day-12: Pub/Sub – Messaging Basics

- Use-case: Async processing in e-commerce
- Create topic and publish messages
- Push vs Pull subscriptions
- Connect Pub/Sub with Cloud Functions (preview)

---

## Day-13: Docker on GCP – Containerizing Apps

- Install Docker on Compute Engine
- Write a simple Node.js/Python app and dockerize it
- Push image to DockerHub or Artifact Registry
- Run container in background with exposed port

---

## Day-14: Artifact Registry – Secure Image Storage

- Setup private Docker repository
- Push/pull from GCE and GKE
- Image tagging and versioning strategy (dev, staging, prod)
- Clean-up policies and access control

---

## Day-15: Intro to Kubernetes + GKE Cluster Setup

- Kubernetes basics (pods, deployments, services)
- Create a GKE cluster with 2 nodes
- Access via `kubectl` and explore dashboard
- Deploy basic NGINX or app container

---

## Day-16: Deploy a Real App on GKE

- Push your custom Docker app to Artifact Registry
- Deploy app using `Deployment.yaml` and `Service.yaml`
- Expose using LoadBalancer service
- Rolling update and rollback demo

---

## Day-17: Cloud Build – CI/CD 

- Setup Cloud Build with GitHub trigger
- Create `cloudbuild.yaml` file
- Auto-build Docker image and push to registry
- Bonus: Deploy to GKE as part of pipeline

---

## Day-18: Terraform – Infrastructure as Code

- Install Terraform and configure for GCP
- Write `.tf` to create VM, bucket, and firewall
- Store state file in GCS
- Destroy infra using single command (clean-up demo)

---

## Day-19: Real-Time DevOps Project

- Use Terraform to provision infra
- Use Docker + Cloud Build for app containerization
- Store secrets securely
- Use GKE for deployment and Pub/Sub for async task
- Enable monitoring and alerts

---

## Day-20: Serverless with Cloud Run and Functions

- Intro to serverless architecture – when to use
- Deploy container to **Cloud Run** with autoscaling
- Create a small API using **Cloud Functions**
- Compare cost and simplicity vs GKE
- Real-life use-case: background jobs, webhooks

---

## Day-21: GCP Cost Optimization for DevOps

- Understand billing breakdown: Compute, Network, Storage
- Tips to reduce cost:
  - Use preemptible VMs
  - Use autoscaling for GKE and Cloud Run
  - Delete unused disks, snapshots, IPs
- Setup budget alerts and detailed reporting
- Use Recommender and Policy Analyzer

---


