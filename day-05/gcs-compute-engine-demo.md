# 🚀 Hands-On Demo: Google Cloud Storage with Compute Engine VM

In this demo, we will connect **Google Cloud Storage (GCS)** with a **Compute Engine VM** using the **Google Cloud Console (UI only)**.  
You will learn how to create a bucket, set up IAM with a service account, launch a VM, and securely access storage from inside the VM.

---

## 🔑 Learning Objectives
By completing this demo, you will:
- Create a **Cloud Storage bucket**.  
- Create and configure a **Service Account** for secure access.  
- Launch a **Compute Engine VM** with the Service Account attached.  
- Use the VM to **read and write objects** in the bucket.  
- Clean up resources when finished.  

---

## 📝 Step-by-Step Instructions

### **Step 1: Create a Cloud Storage Bucket**
1. Go to the **Google Cloud Console** → **Navigation Menu (☰) → Cloud Storage → Buckets**.  
2. Click **Create bucket**.  
3. Enter a **unique name** (e.g., `gcs-demo-bucket-2025`).  
   - Bucket names must be globally unique across GCP.  
4. Choose **Location type → Region** and pick a region close to you (e.g., `asia-south1 (Mumbai)`).  
5. Leave the **Storage class** as **Standard**.  
6. Under **Access control**, keep it at **Uniform** (recommended).  
7. Click **Create**.  

✅ You now have a storage bucket ready.  

---

### **Step 2: Create a Service Account**
1. Navigate to **Navigation Menu → IAM & Admin → Service Accounts**.  
2. Click **Create Service Account**.  
3. Enter the name: `gcs-demo-sa`.  
4. Click **Create and continue**.  
5. In **Grant this service account access to project**, select the role:  
   - **Storage → Storage Object Admin** (allows the VM to read/write bucket objects).  
6. Click **Continue → Done**.  

✅ A service account is now available for attaching to your VM.  

---

### **Step 3: Launch a Compute Engine VM**
1. Go to **Navigation Menu → Compute Engine → VM Instances**.  
2. Click **Create Instance**.  
3. Configure:
   - **Name:** `gcs-demo-vm`.  
   - **Region/Zone:** Same as your bucket (for lower latency).  
   - **Machine type:** `e2-micro` (eligible for free tier).  
   - **Boot disk:** Keep the default (Debian/Ubuntu).  
4. Scroll down to **Identity and API access**:
   - Under **Service account**, choose `gcs-demo-sa`.  
   - For **Access scopes**, choose **Allow full access to all Cloud APIs** (simpler for this demo).  
5. Click **Create**.  

✅ You now have a VM that can access GCS without any keys.  

---

### **Step 4: Upload a File to Your Bucket**
1. Go to **Cloud Storage → Buckets → your bucket name**.  
2. Click **Upload Files**.  
3. Select a local file (e.g., `hello.txt`).  
4. Verify that the file is visible in the bucket.  

✅ This file will be accessed from the VM.  

---

### **Step 5: Access the Bucket from the VM**
1. Go to **Compute Engine → VM instances**.  
2. Click **SSH** next to your VM (`gcs-demo-vm`) to open a browser terminal.  
3. Inside the VM terminal, run the following commands:  

   - **List files in your bucket:**
     ```
     gcloud storage ls gs://<your-bucket-name>
     ```

   - **Download the uploaded file:**
     ```
     gcloud storage cp gs://<your-bucket-name>/hello.txt hello-vm.txt
     cat hello-vm.txt
     ```

   - **Upload a new file back to GCS:**
     ```
     echo "Hello from Compute Engine VM" > vm-upload.txt
     gcloud storage cp vm-upload.txt gs://<your-bucket-name>/vm-upload.txt
     ```

4. Go back to the **Cloud Storage Console** and confirm that `vm-upload.txt` is present in your bucket.  

✅ The VM can now read and write objects to GCS using its service account.  

---

### **Step 6: Cleanup Resources**
When done, clean up to avoid charges:  
1. Delete the **VM** → Go to **Compute Engine → VM Instances → Select VM → Delete**.  
2. Delete the **Bucket** → Go to **Cloud Storage → Buckets → Select Bucket → Delete**.  
3. Delete the **Service Account** → Go to **IAM & Admin → Service Accounts → Delete**.  
4. If this project was created just for testing, you can delete the **entire project**, which removes all resources.  

---

## 🎯 Key Takeaways
- **Cloud Storage** is the default choice for storing objects like logs, backups, and artifacts.  
- **Service Accounts** provide **secure, keyless authentication** for applications and VMs.  
- **Compute Engine VMs** can interact with GCS easily when IAM roles are attached.  
- Always **clean up resources** after hands-on work to avoid unexpected billing.  

