# Ultimate Kubernetes project on GKE

In this Kubernetes project, we will learn how to deploy a multi microservice architectured application.

- Code for the application
https://github.com/dockersamples/example-voting-app

<img width="860" height="800" alt="architecture excalidraw" src="https://github.com/user-attachments/assets/581a965f-3185-4f30-902f-0f3232d1fe9a" />


### Step 1: Enable Container API

```
gcloud services enable container.googleapis.com
```

### Step 2: Create a GKE Cluster

- Create a HA GKE cluster with 3 nodes
```
gcloud container clusters create my-cluster \
  --region us-central1 \
  --num-nodes 3 \
  --machine-type e2-standard-4
```

- Create GKE cluster with Autopilot Mode (Google manages the nodes for you)
```
gcloud container clusters create-auto my-autopilot-cluster \
  --region us-central1
```

### Step 3: Connect to the cluster from Cloudshell or VM

```
gcloud container clusters get-credentials autopilot-cluster --region us-central1
```

