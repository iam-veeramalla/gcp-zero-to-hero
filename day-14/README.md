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

### Step 4: Clone the Git repo

```
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app
```

### Step 5: Run the app on the cluster

```
kubectl apply -f k8s-specifications/
```

Verify
```
kubectl get all
```

### Step 6: Access app using Nodeport

```
kubectl port-forward svc/vote 8080:8080
```

### Step 7: Expose using Ingress

Step 1: Create Ingress resource
```
kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.service.type=LoadBalancer
```

Step 2: Deploy Voting app Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vote
            port:
              number: 8080
```

Step 3: Deploy Result app Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: result-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /result
        pathType: Prefix
        backend:
          service:
            name: result
            port:
              number: 8081
```

Step 4: Verify Ingress and access the apps

```
kubectl get ingress
```
