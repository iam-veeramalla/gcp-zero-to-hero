# Day-12 Hands-On: Google Cloud Artifact Registry

> End goal: From **Cloud Shell**, create an **Artifact Registry (AR)** repo and a **Service Account (SA)**.  
> Then create a **VM with that SA** attached.  
> **Connect using the Console “SSH” button** (not `gcloud compute ssh`), set env vars **on the VM**, install Docker + gcloud, build a **hello-world** image, and **push** it to AR using the VM’s SA.

---

## 0) One-time setup in **Cloud Shell**

    export PROJECT_ID="my-gcp-project"
    export REGION="asia-south1"          # closest region to your compute
    export ZONE="asia-south1-a"
    export REPO="apps"                   # AR repository name
    export IMAGE="hello"                 # your app/service name
    export TAG="v1.0.0"                  # or dev-2025-09-08, etc.
    export AR_HOST="${REGION}-docker.pkg.dev"
    export IMG_PATH="${AR_HOST}/${PROJECT_ID}/${REPO}/${IMAGE}"

    gcloud config set project ${PROJECT_ID}

    # Enable required APIs (idempotent)
    gcloud services enable artifactregistry.googleapis.com compute.googleapis.com iam.googleapis.com

---

## 1) Create a **Docker repo** in Artifact Registry (Cloud Shell)

    gcloud artifacts repositories create ${REPO} \
      --repository-format=docker \
      --location=${REGION} \
      --description="App container images"

    # Verify
    gcloud artifacts repositories describe ${REPO} --location=${REGION}

---

## 2) Create a **Service Account** with least privilege (Cloud Shell)

    gcloud iam service-accounts create ar-pusher \
      --display-name="Artifact Registry Pusher"

    # Grant writer role on THIS repo (scoped to repo, not project-wide)
    gcloud artifacts repositories add-iam-policy-binding ${REPO} \
      --location=${REGION} \
      --member="serviceAccount:ar-pusher@${PROJECT_ID}.iam.gserviceaccount.com" \
      --role="roles/artifactregistry.writer"

---

## 3) Create a **VM** with the Service Account attached (Cloud Shell)

    gcloud compute instances create ar-builder \
      --zone=${ZONE} \
      --machine-type=e2-micro \
      --service-account=ar-pusher@${PROJECT_ID}.iam.gserviceaccount.com \
      --scopes=https://www.googleapis.com/auth/cloud-platform

> This attaches the SA to the VM so it can get access tokens via the metadata server (no key files).

---

## 4) **Connect to the VM using Console → “SSH” button**

- In the GCP Console, go to **Compute Engine → VM instances → ar-builder → SSH**.
- You are now inside the VM shell.

**Set env vars on the VM** (so the rest of the steps work from here):

    # Auto-detect PROJECT_ID and ZONE from metadata (recommended)
    export PROJECT_ID="$(curl -H 'Metadata-Flavor: Google' -s http://metadata.google.internal/computeMetadata/v1/project/project-id)"
    export ZONE="$(curl -H 'Metadata-Flavor: Google' -s http://metadata.google.internal/computeMetadata/v1/instance/zone | awk -F/ '{print $NF}')"
    export REGION="${ZONE%-*}"

    # Set the same values used earlier
    export REPO="apps"
    export IMAGE="hello"
    export TAG="v1.0.0"
    export AR_HOST="${REGION}-docker.pkg.dev"
    export IMG_PATH="${AR_HOST}/${PROJECT_ID}/${REPO}/${IMAGE}"

    # Point gcloud to the right project (we'll install gcloud next)
    echo "${PROJECT_ID}"

---

## 5) Install **Docker** and **gcloud CLI** on the VM

    # Update apt and install Docker
    sudo apt-get update -y
    sudo apt-get install -y docker.io ca-certificates gnupg

    # Allow your user to run Docker without sudo
    sudo usermod -aG docker $USER
    newgrp docker

    # Install Google Cloud CLI (official repo)
    echo "Installing gcloud CLI..."
    sudo install -m 0755 -d /usr/share/keyrings
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
      | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
      | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y google-cloud-cli

    # Set project in gcloud (uses the attached SA when on GCE)
    gcloud config set project ${PROJECT_ID}

    # Sanity check: should print an access token via the VM's service account
    gcloud auth print-access-token | head -c 20 && echo " ...ok"

---

## 6) Create a **Hello World** image (on the VM)

    # Minimal Dockerfile
    cat > Dockerfile <<'EOF'
    FROM nginx:1.27-alpine
    RUN echo 'Hello from Artifact Registry!' > /usr/share/nginx/html/index.html
    EOF

    # Build locally
    docker build -t ${IMAGE}:local .

    # Test run
    docker run -d --rm -p 8080:80 ${IMAGE}:local
    curl -sS http://127.0.0.1:8080 | head -n1
    # Stop the container
    docker ps -q --filter "ancestor=${IMAGE}:local" | xargs -r docker stop

---

## 7) **Push** the image to Artifact Registry using the VM’s Service Account

### Option A (recommended): Configure Docker to use the **gcloud** credential helper

    # Configure Docker auth for your AR host (writes to ~/.docker/config.json)
    gcloud auth configure-docker ${AR_HOST}

    # Tag for AR and push
    docker tag ${IMAGE}:local ${IMG_PATH}:${TAG}
    docker push ${IMG_PATH}:${TAG}

### Option B (fallback): Use an **access token** with `docker login`

    docker login -u oauth2accesstoken -p "$(gcloud auth print-access-token)" https://${AR_HOST}
    docker tag ${IMAGE}:local ${IMG_PATH}:${TAG}
    docker push ${IMG_PATH}:${TAG}

**Verify from the VM**

    gcloud artifacts docker images list ${AR_HOST}/${PROJECT_ID}/${REPO} --include-tags

**(Optional) Promote by digest without rebuilding**

    docker pull ${IMG_PATH}:${TAG}
    docker inspect --format='{{index .RepoDigests 0}}' ${IMG_PATH}:${TAG}
    # Example: asia-south1-docker.pkg.dev/PROJECT/REPO/IMAGE@sha256:abcd...

    DIGEST="<copy the full sha256 ref above>"
    docker pull "${DIGEST}"
    for env in dev staging prod; do
      docker tag "${DIGEST}" "${IMG_PATH}:${env}"
      docker push "${IMG_PATH}:${env}"
    done

---

## Troubleshooting (on the VM)

    # Permission denied (push/pull):
    # - Ensure VM is using SA: ar-pusher@${PROJECT_ID}.iam.gserviceaccount.com
    # - Ensure repo-level role: roles/artifactregistry.writer for that SA
    # - Re-run: gcloud auth configure-docker ${AR_HOST}

    # "no basic auth credentials":
    # - Use Option B docker login with access token OR re-run Option A

    # "manifest unknown":
    # - Check region/host, repo, image name, and tag
    # - List images:
    #   gcloud artifacts docker images list ${AR_HOST}/${PROJECT_ID}/${REPO} --include-tags

---

## Clean up (optional)

    # From Cloud Shell or VM:
    gcloud compute instances delete ar-builder --zone=${ZONE} --quiet

    # Remove the repo (deletes all images)
    # gcloud artifacts repositories delete ${REPO} --location=${REGION} --quiet
