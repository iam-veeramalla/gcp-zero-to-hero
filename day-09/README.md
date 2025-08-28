# Day-9: Google Cloud CLI (gcloud) Deep Dive

In earlier days we built everything from the **Console (UI)**. From Day-9 onward, we’ll use the **Google Cloud CLI (`gcloud`)** so learners can automate, script, and work like real DevOps/Cloud engineers.

---

## Learning Outcomes

- Understand how `gcloud` maps to the same APIs used by the Console.
- Set up configurations (project, region, zone) and authenticate correctly.
- Create, list, update, and delete core resources (Compute VM, Firewall, GCS bucket) via CLI.

---

## Prerequisites

- A Google Cloud project with billing enabled.
- Console access to verify results created via CLI (optional).
- Practice environment:
  - **Recommended:** Cloud Shell (already has `gcloud` preinstalled).
  - **Local (optional):** Install Cloud SDK → https://cloud.google.com/sdk/docs/install

> Note: `gcloud components update` is not available in Cloud Shell; it’s for local SDK installs.

---

## Quick Start: Initialize and Configure

```bash
gcloud init
gcloud auth list
gcloud config list
```

```
PROJECT_ID="<YOUR_PROJECT_ID>"
REGION="us-central1"
ZONE="us-central1-a"

gcloud config set project ${PROJECT_ID}
gcloud config set compute/region ${REGION}
gcloud config set compute/zone ${ZONE}
```

### Create a VM and Open HTTP with CLI

1. Create a small Debian VM

```
gcloud compute instances create day9-vm \
  --machine-type=e2-micro \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

2. Fetch its external IP

```
gcloud compute instances list --filter="name=day9-vm" --format="value(EXTERNAL_IP)"
```

3. SSH in (Cloud Shell attaches automatically)

```
gcloud compute ssh day9-vm --zone=${ZONE}
```

4. Cleanup 

```
gcloud compute instances delete day9-vm --zone=${ZONE}
```


### Create and Manage a GCS Bucket

1. Pick a unique name and create the bucket

```
BUCKET="day9-bucket-$RANDOM"
gcloud storage buckets create gs://${BUCKET} --location=${REGION}
```

2. Upload a simple file to the bucket

```
echo "hello from day9" > hello.txt
gcloud storage cp hello.txt gs://${BUCKET}/hello.txt
```

3. List and read the object

```
gcloud storage ls gs://${BUCKET}
gcloud storage cat gs://${BUCKET}/hello.txt
```

4. Remove the object and delete the bucket

```
gcloud storage rm gs://${BUCKET}/hello.txt
gcloud storage buckets delete gs://${BUCKET}
```
