---

# GCP Console Setup Guide

Managed Instance Group + Auto-Scaling (80%) + Firewall + IAM

---

## Step 1 — Create / Select Project

1. Go to: [https://console.cloud.google.com](https://console.cloud.google.com)
2. Select your project (top navigation bar).
3. Ensure **Billing is enabled**.

---

## Step 2 — Create VPC Network

1. Navigate to
   **VPC Network → VPC networks**
2. Click **Create VPC network**
3. Choose:

   * Name: `vm-network`
   * Subnet creation mode: **Automatic**
4. Click **Create**

You now have a network with auto-created subnets.

---

## Step 3 — Configure Firewall Rules

Navigate to:

**VPC Network → Firewall**

### A. Allow ICMP

1. Click **Create Firewall Rule**
2. Name: `allow-icmp`
3. Network: `vm-network`
4. Direction: Ingress
5. Action: Allow
6. Targets: All instances in network
7. Source IP ranges: `0.0.0.0/0`
8. Protocols and ports:

   * Select **Specified protocols**
   * Check **ICMP**
9. Click **Create**

---

### B. Allow SSH (Port 22)

1. Click **Create Firewall Rule**
2. Name: `allow-ssh`
3. Network: `vm-network`
4. Direction: Ingress
5. Action: Allow
6. Targets: All instances
7. Source IP ranges: `0.0.0.0/0`
   (For production, restrict this.)
8. Protocols and ports:

   * Check **TCP**
   * Enter port `22`
9. Click **Create**

All other ports remain blocked by default.

Minimal attack surface achieved.

---

## Step 4 — Create Instance Template

Navigate to:

**Compute Engine → Instance templates**

1. Click **Create instance template**

Configure:

Basic:

* Name: `vm-template`
* Region: your region

Machine:

* Series: E2
* Machine type: `e2-medium` (or as required)

Boot disk:

* Change → Debian 12 (or Ubuntu)
* Standard persistent disk

Networking:

* Network: `vm-network`
* Subnet: Auto

Leave everything else default unless needed.

Click **Create**

This template is your immutable VM blueprint.

---

## Step 5 — Create Managed Instance Group (MIG)

Navigate to:

**Compute Engine → Instance groups**

1. Click **Create instance group**
2. Choose:

   * Group type: **New managed instance group (stateless)**
   * Name: `vm-mig`
   * Location type: Zonal (or Regional if you prefer)
   * Zone: choose one
   * Instance template: `vm-template`

### Initial size:

Set to **0**

Click **Create**

You now have a Managed Instance Group with zero instances running.

Cloud minimalism.

---

## Step 6 — Configure Auto-Scaling

Inside your instance group page:

1. Click **Edit**
2. Scroll to **Autoscaling**
3. Select **On**

Configure:

* Autoscaling mode: **On: add and remove instances**
* Minimum number of instances: **0**
* Maximum number of instances: **10**
* Autoscaling metric: **CPU utilization**
* Target CPU utilization: **80%**

Leave cooldown period default (or 60 seconds).

Click **Save**

Now your MIG behaves like:

If avg CPU > 80% → scale up
If CPU low → scale down
Can go down to zero.

Elastic computing unlocked.

---

## Step 7 — Configure Health Check

Navigate to:

**Compute Engine → Health checks**

1. Click **Create Health Check**
2. Name: `vm-health-check`
3. Protocol: HTTP (or TCP if no web server)
4. Port: 80 (or relevant port)
5. Leave defaults
6. Click **Create**

Now attach it:

1. Go back to **Instance Groups**
2. Click `vm-mig`
3. Click **Edit**
4. Under **Health check**, select `vm-health-check`
5. Save

Health checks ensure unhealthy VMs are replaced.

Self-healing infrastructure — distributed Darwinism.

---

## Step 8 — IAM Role Configuration

Navigate to:

**IAM & Admin → IAM**

You need to ensure:

Health Checker role:
Cloud Run Service Agent

If not present:

1. Click **Grant Access**
2. In **New principals**, enter:

```
service-<PROJECT_NUMBER>@serverless-robot-prod.iam.gserviceaccount.com
```

3. Role:
   Search and select:
   **Cloud Run Service Agent**
4. Click **Save**

This ensures health checking and related service communication works correctly.

---

## Step 9 — Confirm Packet Mirroring is NOT Configured

Navigate to:

**VPC Network → Packet Mirroring**

No entries should be present
---
