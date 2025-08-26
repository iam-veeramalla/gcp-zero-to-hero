# Day-8: Private App on Custom VPC + Regional External HTTP(S) Load Balancer + Custom Domain (GoDaddy + Cloud DNS)

## Pre-requisites

- A Google Cloud project with billing enabled
- Basic familiarity with the GCP Console (we use **UI only**; CLI optional for verification)
- A GoDaddy domain you can modify (or be ready to purchase one)
- Region choice (example used below: `asia-south1`). Use one region consistently.

---

## Step 1 — Create the VPC and subnets (Console)

1) In the console, go to: VPC network → VPC networks → Create VPC network
   - Name: prod-vpc
   - Subnet creation mode: Custom
   - Add subnets (same Region you chose):
     - Name: subnet-public, IP range: 10.10.10.0/24
     - Name: subnet-app,    IP range: 10.10.20.0/24
   - Click Create.

2) Create the **proxy-only** subnet (required for Regional External HTTP(S) LB):
   - VPC networks → prod-vpc → Subnetworks → Create subnetwork
   - Name: subnet-proxy-only
   - Region: your region (e.g., asia-south1)
   - IP range: 10.10.30.0/24
   - Purpose: Regional Managed Proxy (proxy-only)
   - Click Create.

---

## Step 2 — Cloud Router and Cloud NAT (private VM egress)

We want private VMs (no external IPs) to reach the internet for apt/pip.

1) Network services → Cloud NAT → Create NAT gateway
   - Name: nat-asia-south1
   - Region: your region
   - Network: prod-vpc
   - Cloud Router: Create new → router-asia-south1
   - NAT IP addresses: Auto-allocate (okay for demo)
   - Subnetworks: Select subnet-app (and subnet-public if you like)
   - Create.

---

## Step 3 — Create firewall rules for health checks and data plane

We need two allows on TCP 8080:

A) Allow **Google health-check** ranges to reach your backends:
   - VPC network → Firewall → Create firewall rule
   - Name: allow-health-checks-8080
   - Direction: Ingress
   - Targets: All instances in the network (or use a network tag from your template)
   - Source IPv4 ranges: 35.191.0.0/16, 130.211.0.0/22
   - Protocols/ports: TCP:8080
   - Create.

B) Allow **LB data-plane** traffic from the proxy-only subnet to your backends:
   - VPC network → Firewall → Create firewall rule
   - Name: allow-proxy-only-8080
   - Direction: Ingress
   - Targets: All instances in the network (or tag)
   - Source IPv4 ranges: 10.10.30.0/24   (your proxy-only subnet CIDR)
   - Protocols/ports: TCP:8080
   - Create.

---

## Step 4 — Create the Instance Template (Flask app via startup script)

We will install Python3, Flask, and Gunicorn, and run on port 8080 with `/health`.

In the Instance Template UI, under “Automation → Startup script”, paste the script below.

```
#!/bin/bash
set -e
apt-get update -y
apt-get install -y python3-pip
useradd -m -s /bin/bash appuser || true
su - appuser -c "pip3 install flask gunicorn"

cat >/home/appuser/app.py <<'PY'
from flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "Hello from Flask on GCP (private subnet)!"

@app.route("/health")
def health():
    return "ok", 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
PY

chown appuser:appuser /home/appuser/app.py

cat >/etc/systemd/system/flask.service <<'UNIT'
[Unit]
Description=Flask via Gunicorn
After=network.target

[Service]
User=appuser
WorkingDirectory=/home/appuser
ExecStart=/usr/bin/gunicorn -w 2 -b 0.0.0.0:8080 app:app
Restart=always

[Install]
WantedBy=multi-user.target
UNIT

systemctl daemon-reload
systemctl enable --now flask.service
```

Create the template:

1) Compute Engine → Instance templates → Create instance template
   - Name: tmpl-flask-app
   - Machine type: e2-micro or e2-small (demo)
   - Boot disk: Debian/Ubuntu (any recent LTS)
   - Networking:
     - Network: prod-vpc
     - Subnet: subnet-app
     - External IP: None (keep private)
   - Firewall section: leave unchecked (we use explicit rules created earlier)
   - Automation: paste the startup script
   - Create.

---

## Step 5 — Create the Managed Instance Group (MIG)

1) Compute Engine → Instance groups → Create instance group
   - Name: mig-flask-app
   - Location: Single zone (e.g., asia-south1-a)
   - Instance template: tmpl-flask-app
   - Autoscaling: Min 2, Max 4 (demo)
   - Create.

> Verification (optional, via a bastion in subnet-public): you can SSH to a bastion with an external IP, then SSH to private instances using internal IP, and curl http://<vm-internal-ip>:8080/health

---

## Step 6 — Create the Regional External HTTP(S) Load Balancer

1) Network services → Load balancing → Create load balancer
   - Choose: Application Load Balancer
   - From Internet to my VMs or serverless services → Start configuration
   - Load balancer scope: Regional
   - Region: your chosen region
   - Continue.

2) Frontend (HTTP for now)
   - Protocol: HTTP
   - Port: 80
   - IP address: Create or select a new External IPv4 (e.g., flask-ip)
   - VPC network for forwarding rule: prod-vpc
   - Save.

3) Backend
   - Backend type: Instance group
   - Instance group: mig-flask-app
   - Port: 8080
   - Health check: Create new → HTTP on port 8080 → Request path /health
   - Save.

4) Routing
   - Default URL map → points to the backend above

5) Proxy-only subnet
   - Ensure your Regional LB references your proxy-only subnet in the same region/VPC (we created it in Step 1).

6) Review & Create
   - After provisioning, open the frontend IP in a browser:
     - You should see: Hello from Flask on GCP (private subnet)!

---

## Step 7 — Domain setup (GoDaddy high-level) + Cloud DNS records

High-level in GoDaddy:
- Buy or use an existing domain (e.g., example.in)
- In the domain’s DNS/Nameservers panel, choose “Custom nameservers”
- You will set these to the 4 nameservers that Google gives you in Cloud DNS (next steps)

Create a Cloud DNS public zone:
1) Network services → Cloud DNS → Create zone
   - Zone type: Public
   - Zone name: example-in-zone (any friendly name)
   - DNS name: example.in.
   - Create.
   - Copy the 4 nameservers (ns-cloud-*.googledomains.com).

Add records in Cloud DNS (after the LB is ready and has an external IP):
1) Add an A record at the apex:
   - Add standard → Type: A
   - Name: (leave blank or use @)
   - TTL: 300
   - IPv4 address: <your LB external IPv4>
   - Save.

2) Optional CNAME for www:
   - Type: CNAME
   - Name: www
   - Canonical name: example.in.
   - TTL: 300
   - Save.

Delegate the domain to Google in GoDaddy:
- In GoDaddy → Nameservers → set to the 4 Cloud DNS nameservers you copied
- Save. Propagation can take some time (usually minutes, sometimes longer)

---

## Step 8 — Validate

By IP (before DNS):
- Open http://<LB_IP>/
- Open http://<LB_IP>/health  → should return ok

By domain (after delegation + A record):
- Open http://example.in/
- Open http://example.in/health  → should return ok

Optional terminal checks (any machine with dig/curl):
- dig A example.in +short
- dig NS example.in +short
- curl -i http://example.in/

---
