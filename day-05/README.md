# Day-5: Google Cloud Storage services

1. **Cloud Storage (GCS)** – Object storage for unstructured data like logs, artifacts, and backups.  
2. **Filestore** – Managed **NFS file system** for workloads that need shared POSIX file access.  
3. **Local SSDs** – High-performance ephemeral block storage directly attached to VM hosts.  
4. **Bigtable** – Wide-column NoSQL database for large-scale, time-series, or operational data.  
5. **Pub/Sub** – Messaging and event streaming service to connect microservices and trigger workflows.  

---

## 🔹 Key Differences

| Service           | Type              | Scope / Usage                        | Durability & Availability   | Example Equivalent |
|-------------------|------------------|---------------------------------------|-----------------------------|--------------------|
| **Cloud Storage** | Object Storage   | Regional / Multi-regional buckets     | 99.999999999% durability    | AWS S3, Azure Blob |
| **Filestore**     | File (NFS)       | Shared file systems for VMs / GKE     | Regional (zonal redundancy) | AWS EFS, Azure Files |
| **Local SSD**     | Block (Ephemeral)| Ultra-fast temp storage tied to VM    | Tied to VM lifecycle        | AWS Instance Store |
| **Bigtable**      | NoSQL DB         | Wide-column DB for analytics/monitoring | Regional replication      | AWS DynamoDB (similar) |
| **Pub/Sub**       | Messaging Bus    | Event-driven, async service communication | Global service, HA managed | AWS SQS/SNS, Kafka |

---

## 🔹 DevOps Use Cases

### **Cloud Storage (GCS)**
- Store CI/CD artifacts, Helm charts, and Terraform state files.  
- Centralized logging and backup storage for Kubernetes clusters.  
- Host static websites and internal documentation.  

### **Filestore**
- Shared configuration files for stateful workloads in Kubernetes (e.g., Jenkins home).  
- Persistent shared storage for legacy apps lifted into GKE.  

### **Local SSD**
- High-performance cache for CI/CD build pipelines.  
- Temporary data processing (e.g., ETL staging area).  

### **Bigtable**
- Long-term observability data (metrics, logs, traces).  
- Backend for time-series monitoring tools.  

### **Pub/Sub**
- **Event-driven DevOps**: trigger pipelines on code pushes or infra changes.  
- **Log & metrics streaming**: pipe logs from GKE/VMs to BigQuery or GCS.  
- **Incident automation**: integrate alerts into Pub/Sub → trigger Cloud Functions or send to PagerDuty.  
- **Decoupling microservices**: async communication between independent services.  

---

# 🚀 Demo: Google Cloud Storage with Compute Engine VM

This demo shows how to securely connect **Google Cloud Storage (GCS)** with a **Compute Engine VM** using only the **Google Cloud Console (UI)**.  
By completing it, you’ll practice secure, keyless access to cloud storage.

---

## 🔑 What You Will Learn
- Create and configure a **GCS bucket**.  
- Use **service accounts** for secure IAM-based access.  
- Launch a **Compute Engine VM** with an attached service account.  
- Access GCS from inside the VM (read/write files).  
- Safely **clean up resources** after the demo.  

---

## 📝 Step-by-Step Summary

1. **Create a GCS Bucket**
   - Go to **Cloud Storage → Buckets**.  
   - Create a bucket with a unique name and regional location.  
   - Upload a sample file (e.g., `hello.txt`).  

2. **Create a Service Account**
   - Go to **IAM & Admin → Service Accounts**.  
   - Create a new service account (`gcs-demo-sa`).  
   - Assign the **Storage Object Admin** role.  

3. **Launch a Compute Engine VM**
   - Go to **Compute Engine → VM Instances**.  
   - Create a VM (`gcs-demo-vm`) in the same region.  
   - Attach the service account for GCS access.  

4. **Access GCS from the VM**
   - SSH into the VM from the console.  
   - Use `gcloud storage ls` to list bucket files.  
   - Download and upload files between VM and bucket.  

5. **Cleanup**
   - Delete the **VM**.  
   - Delete the **Bucket**.  
   - Delete the **Service Account**.  
   - (Or delete the entire project if it was created just for this demo).

---
