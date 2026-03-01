# SETUP GUIDE FOR CLOUD POLICIES
---

# 1. VM Setup with Auto Scaling

## Step 1: Create Instance Template

1. Open Google Cloud Console
2. Navigate to: Compute Engine → Instance Templates
3. Click Create Instance Template

Configure:

* Machine type: e2-medium (or as required)
* Boot disk: Default OS image
* Network: default VPC
* Leave subnet as default (auto mode)

Firewall:

* Allow HTTP (if needed)
* Allow HTTPS (if needed)

Click Create.

---

## Step 2: Create Managed Instance Group

1. Go to Compute Engine → Instance Groups
2. Click Create Instance Group
3. Select:

   * Type: Managed
   * Instance Template: Select the template created above
   * Location: Single zone

Auto Scaling Configuration:

* Minimum instances: 0
* Maximum instances: 10
* Autoscaling metric: CPU utilization
* Target CPU utilization: 80%

Create the instance group.

Behavior:

If CPU usage exceeds 80%, instances scale up.
If usage drops, instances scale down (minimum 0).

---

# 2. Network Firewall Policy (Geolocation-Based)

This uses Network Firewall Policies under Network Security.

## Step 1: Create Firewall Policy

1. Go to Network Security → Firewall Policies
2. Click Create Firewall Policy
3. Name: india-geo-policy
4. Create

---

## Step 2: Add Geolocation Rule 

Open the created policy → Click Add Rule.

Configure:

General:

* Priority: 1010
* Direction: Ingress
* Action: Allow
* Enable rule

Target:

* Target type: Instances
* Apply to: All instances (or specific secure tag if used)

Source:

* Source network context: All network contexts
* IP type: IPv4
* Under Source filters → Geolocations
* Select: India

Create rule.

This means:

If traffic originates from India → It is forwarded to a specific IP.

---

# 3. Forwarding to Designated Destination IP

Forwarding means traffic is directly delivered to the VM’s external IP.

Under the geolocation there is a destination IP , configure it to the address you want to send to.

Traffic Flow:

Internet
↓
Network Firewall Policy (Priority 1010 – India Allowed)
↓
VM External IP (Designated Destination)

No mirroring is configured.

---

# 4. IAM Role Configuration

Health Checker Role:

Assign:

Cloud Run Service Agent

Steps:

1. Go to IAM & Admin → IAM
2. Locate Cloud Run Service Agent
3. Ensure required permissions are granted

Role: Health Checker 

---
