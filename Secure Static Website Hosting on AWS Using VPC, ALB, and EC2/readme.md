# AWS VPC Static Website — Free Tier Project

> **VPC · Public & Private Subnets · Internet Gateway · Route Tables · Application Load Balancer · EC2 · static website**
> Production-grade, highly available static website on AWS using Free Tier-eligible resources.

---

##  Project Info

| Field | Details |
|---|---|
| **Candidate** | Apraveen |
| **Email** | apraveen1103@gmail.com |
| **AWS Region** | us-east-1 (N. Virginia) |
| **Project Date** | May 2026 |
| **Instance Type** | t2.micro · Amazon Linux 2023 |
| **ALB DNS** | `my-alb-xxxx.us-east-1.elb.amazonaws.com` |

---

##  Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [Step 1 — Create the VPC](#3-step-1--create-the-vpc)
4. [Step 2 — Create Subnets](#4-step-2--create-subnets)
5. [Step 3 — Internet Gateway](#5-step-3--internet-gateway)
6. [Step 4 — Route Tables](#6-step-4--route-tables)
7. [Step 5 — Security Groups](#7-step-5--security-groups)
8. [Step 6 — Launch EC2 Instances](#8-step-6--launch-ec2-instances)
9. [Step 7 — Application Load Balancer](#9-step-7--application-load-balancer)
10. [Step 8 — End-to-End Verification](#10-step-8--end-to-end-verification)
11. [Project Results](#11-project-results)
12. [Skills Demonstrated](#12-skills-demonstrated)

---

## 1. Project Overview

This project demonstrates the design and deployment of a **production-grade, highly available static website** on AWS using Free Tier-eligible resources. The solution uses a custom VPC with public and private subnets, an Internet Gateway, separate route tables, security groups, and an **Application Load Balancer** to distribute HTTP traffic across two EC2 instances hosting an DIFFERENT static site.

### 1.1 Objectives

- Deploy a publicly accessible static website via **ALB DNS endpoint**
- Isolate compute resources in a **private subnet** — no direct internet access
- Use **public subnet + IGW + route table** for correct traffic routing
- Distribute traffic across multiple EC2 instances using **ALB round-robin**
- Keep all infrastructure within the **AWS Free Tier** cost boundary

### 1.2 Architecture Summary

| Component | Details |
|---|---|
| **VPC** | 10.0.0.0/16 · DNS hostnames enabled |
| **Public Subnet** | 10.0.1.0/24 · AZ us-east-1a · Auto-assign public IP ON |
| **Private Subnet** | 10.0.2.0/24 · AZ us-east-1a · No public IP |
| **Internet Gateway** | Attached to VPC — enables public subnet internet access |
| **Public Route Table** | 0.0.0.0/0 → IGW |
| **Private Route Table** | Local only (no IGW route) |
| **ALB Security Group** | Inbound HTTP:80 from 0.0.0.0/0 |
| **EC2 Security Group** | Inbound HTTP:80 from ALB SG only |
| **EC2 Instances** | 2× t2.micro (ec2-web-1, ec2-web-2) in private subnet |
| **Load Balancer** | Application LB · internet-facing · HTTP:80 listener |
| **Web Server** |  static HTML via html FILE |

### 1.3 Free Tier Limits

| Resource | Free Tier |
|---|---|
| VPC, Subnets, IGW, Route Tables | Always free |
| EC2 t2.micro | 750 hrs/month (12 months) |
| Application Load Balancer | 750 hrs/month + 15 LCU-hrs/month (12 months) |
| Data Transfer Out | 1 GB/month free |

---

## 2. Architecture Diagram

```
           [ Internet / Users ]
                    │
                    ▼
          [ Internet Gateway ]        ← attached to VPC (my-static-vpc)
                    │
                    ▼
         [ Public Route Table ]       ← 0.0.0.0/0 → IGW
                    │
┌──────────── PUBLIC SUBNET 10.0.1.0/24 (us-east-1a) ────────────┐
│                                                                │
│          [ Application Load Balancer ]                         │
│            HTTP:80  |  internet-facing                         │
│            Security Group: alb-sg                              │
│                                                                │
└──────────────────────┬─────────────────────────────────────────┘
                       │  ALB distributes traffic
              ┌────────┴────────┐
              ▼                 ▼
┌─────── PRIVATE SUBNET 10.0.2.0/24 (us-east-1a) ────────────────┐
│                                                                │
│   [ EC2: ec2-web-1 ]       [ EC2: ec2-web-2 ]                  │
│     t2.micro                 t2.micro                          │
│      static site             static site                       |
│     No public IP             No public IP                      │
│     Security Group: ec2-sg   Security Group: ec2-sg            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

> **Traffic Flow:** User → Internet → IGW → ALB (public subnet) → EC2 instances (private subnet)
> EC2 instances have **no direct internet access** — only the ALB can reach them.

---

## 3. Step 1 — Create the VPC

### Navigate to: `AWS Console → VPC → Your VPCs → Create VPC`

| Setting | Value |
|---|---|
| **Name tag** | my-static-vpc |
| **IPv4 CIDR block** | 10.0.0.0/16 |
| **IPv6 CIDR block** | No IPv6 |
| **Tenancy** | Default |
| **DNS hostnames** |  Enable |
| **DNS resolution** | Enable |

>  **Important:** Enabling DNS hostnames is required for the ALB to resolve EC2 targets by DNS name.

###  Proof Screenshot



![VPC Created](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/VPC_only.jpg)

*Caption: VPC `my-static-vpc` created with CIDR 10.0.0.0/16 — State: Available*

---
a
## 4. Step 2 — Create Subnets

### Navigate to: `VPC → Subnets → Create Subnet`

#### 4.1 Public Subnet

| Setting | Value |
|---|---|
| **Subnet name** | public-subnet-1a |
| **VPC** | my-static-vpc |
| **Availability Zone** | us-east-1a |
| **IPv4 CIDR block** | 10.0.1.0/24 |
| **Auto-assign public IP** |  Enable (Actions → Modify subnet settings) |

#### 4.2 Private Subnet

| Setting | Value |
|---|---|
| **Subnet name** | private-subnet-1a |
| **VPC** | my-static-vpc |
| **Availability Zone** | us-east-1a |
| **IPv4 CIDR block** | 10.0.2.0/24 |
| **Auto-assign public IP** |  Leave disabled |

> ℹ **Note:** EC2 instances in the private subnet will have no public IP and cannot be reached directly from the internet.



![Public Subnet Created](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/PUB_SUB.jpg)

*Caption: `public-subnet-1a` — CIDR 10.0.1.0/24 — Auto-assign public IP: Yes*

![Private Subnet Created](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/PRV_SUB.jpg)

*Caption: `private-subnet-1a` — CIDR 10.0.2.0/24 — Auto-assign public IP: No*

---

## 5. Step 3 — Internet Gateway

### Navigate to: `VPC → Internet Gateways → Create Internet Gateway`

**Step-by-step:**

1. Set **Name tag:** `my-igw` → click **Create Internet Gateway**
2. Select the new IGW → **Actions → Attach to VPC**
3. Select `my-static-vpc` → click **Attach Internet Gateway**

| Field | Value |
|---|---|
| **IGW Name** | my-igw |
| **State after creation** | Detached |
| **State after attach** | Attached |
| **Attached VPC** | my-static-vpc |

>  **Important:** An IGW can only be attached to one VPC at a time. Without this, the public subnet has no internet access and the ALB will not be reachable.

### Proof Screenshot — IGW Created



![IGW Created](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/IGW.jpg)


*Caption: `my-igw` attached to `my-static-vpc` — State: Attached*

---

## 6. Step 4 — Route Tables

### Navigate to: `VPC → Route Tables → Create Route Table`

#### 6.1 Public Route Table

**Step-by-step:**

1. Create route table → **Name:** `public-rt` | **VPC:** `my-static-vpc` → Create
2. Select `public-rt` → **Routes tab → Edit routes → Add route:**

| Destination | Target | Purpose |
|---|---|---|
| 10.0.0.0/16 | local | VPC-internal traffic (auto-added) |
| 0.0.0.0/0 | my-igw | All outbound traffic → Internet |

3. **Subnet Associations tab → Edit subnet associations** → select `public-subnet-1a` → Save

#### 6.2 Private Route Table

**Step-by-step:**

1. Create route table → **Name:** `private-rt` | **VPC:** `my-static-vpc` → Create
2. **Routes tab** — keep only the default `local` route. **Do NOT add an IGW route.**
3. **Subnet Associations tab → Edit subnet associations** → select `private-subnet-1a` → Save

| Destination | Target | Purpose |
|---|---|---|
| 10.0.0.0/16 | local | VPC-internal traffic only |

> **Security Note:** No IGW route on the private route table means EC2 instances in the private subnet have **zero outbound internet access**. They are only reachable from within the VPC (i.e. from the ALB).



![Public Route Table](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/PUB_RT.jpg)

*Caption: `public-rt` — Routes show 0.0.0.0/0 → my-igw. Associated with public-subnet-1a*


![Private Route Table](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/PRV_RT.jpg)

*Caption: `private-rt` — Only local route. Associated with private-subnet-1a*

---

## 7. Step 5 — Security Groups

### Navigate to: `EC2 → Security Groups → Create Security Group`

Two security groups are required — one for the ALB (public-facing) and one for the EC2 instances (accepts traffic from ALB only).

#### 7.1 ALB Security Group (`alb-sg`)

| Field | Value |
|---|---|
| **Name** | alb-sg |
| **Description** | Allow HTTP from internet to ALB |
| **VPC** | my-static-vpc |

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 | Allow all HTTP from internet |

**Outbound Rules:** All traffic (default)

#### 7.2 EC2 Security Group (`ec2-sg`)

| Field | Value |
|---|---|
| **Name** | ec2-sg |
| **Description** | Allow HTTP only from ALB |
| **VPC** | my-static-vpc |

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| SSH | TCP | 22 | My IP | Admin access only |
| HTTP | TCP | 80 | **alb-sg** (SG ID) | Traffic from ALB only |

> **Key Security Practice:** Using the ALB Security Group ID as the HTTP source means EC2 instances are **only reachable through the ALB** — not directly from the internet.

![ALB Security Group](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/PUB_SG.jpg)

*Caption: `alb-sg` inbound rules — HTTP:80 from 0.0.0.0/0*

![EC2 Security Group](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/PRV_SG.jpg)

*Caption: `ec2-sg` inbound rules — HTTP:80 source is `alb-sg` SG ID (not 0.0.0.0/0)*

---

## 8. Step 6 — Launch EC2 Instances

### Navigate to: `EC2 → Instances → Launch Instances`

Launch **two instances** — `ec2-web-1` and `ec2-web-2` — with identical settings below.

#### 8.1 Instance Configuration

| Setting | Value |
|---|---|
| **Name** | ec2-web-1 (repeat for ec2-web-2) |
| **AMI** | Amazon Linux 2023 (Free Tier eligible) |
| **Instance type** | t2.micro (Free Tier eligible) |
| **Key pair** | Select or create a .pem key pair |
| **VPC** | my-static-vpc |
| **Subnet** | private-subnet-1a |
| **Auto-assign public IP** | Disable |
| **Security group** | ec2-sg |
| **Storage** | 8 GiB gp2 (default — Free Tier) |

#### 8.3 Verify Instance Status

Wait for both instances to show:
- **Instance state:** `Running`
- **Status checks:** `3/3 checks passed`


![EC2 Running](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/EC2.jpg)

*Caption: `EC2-INSTANCE` — State: Running — Status checks: 2/2 passed — No public IP*

---

## 9. Step 7 — Application Load Balancer

### Navigate to: `EC2 → Load Balancers → Create Load Balancer → Application Load Balancer`

#### 9.1 Create Target Group (do this first)

### Navigate to: `EC2 → Target Groups → Create Target Group`

**Step-by-step:**

1. **Target type:** Instances
2. **Target group name:** `my-tg`
3. **Protocol:** HTTP | **Port:** 80
4. **VPC:** my-static-vpc
5. **Health check path:** `/`
6. **Healthy threshold:** 2 | **Interval:** 30s → click **Next**
7. **Register targets:** select `ec2-web-1` and `ec2-web-2` → **Include as pending below**
8. Click **Create target group**

| Setting | Value |
|---|---|
| **Name** | my-tg |
| **Target type** | Instances |
| **Protocol / Port** | HTTP / 80 |
| **VPC** | my-static-vpc |
| **Health check path** | / |
| **Healthy threshold** | 2 |
| **Unhealthy threshold** | 2 |
| **Interval** | 30 seconds |
| **Registered targets** | ec2-web-1, ec2-web-2 |

![Target Group Created](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/TRG.jpg)

*Caption: `my-tg` — Protocol: HTTP:80 — Targets: ec2-web-1, ec2-web-2 — Health: Healthy*

---

#### 9.2 Create the Application Load Balancer

**Step-by-step:**

1. **Load balancer name:** `my-static-alb`
2. **Scheme:** Internet-facing
3. **IP address type:** IPv4
4. **VPC:** my-static-vpc
5. **Mappings:** us-east-1a → select `public-subnet-1a`
6. **Security groups:** remove default → add `alb-sg`
7. **Listeners and routing:** HTTP:80 → Forward to `my-tg`
8. Click **Create load balancer**

| Setting | Value |
|---|---|
| **Name** | my-static-alb |
| **Scheme** | Internet-facing |
| **IP address type** | IPv4 |
| **VPC** | my-static-vpc |
| **Availability Zone** | us-east-1a → public-subnet-1a |
| **Security group** | alb-sg |
| **Listener** | HTTP : 80 |
| **Default action** | Forward to: my-tg |

> Wait ~2-3 minutes for the ALB **State** to change from `provisioning` → `active`.

>  **Free Tier:** ALB gives you 750 hours/month and 15 LCU-hours/month free for 12 months.

![ALB Active](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/ALB.jpg)

*Caption: `my-static-alb` — State: Active — Scheme: Internet-facing — DNS name copied*


## 10. Step 8 — End-to-End Verification
```
-

![Website ec2-web-1](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/result_2.jpg)

*Caption: Browser shows "Hello from ec2-web-1" via ALB DNS — HTTP 200 OK*



![Website ec2-web-2](https://github.com/praveen110399/Aws-project/blob/9b505e3857a3b10d28578f86dba477dcb19cbec6/VPC/result_1.jpg)

---

## 11. Project Results

| Milestone | Status |
|---|---|
| VPC created with CIDR 10.0.0.0/16 |  Confirmed |
| Public subnet (10.0.1.0/24) — auto public IP ON | Confirmed |
| Private subnet (10.0.2.0/24) — no public IP |  Confirmed |
| Internet Gateway created and attached to VPC |  Confirmed |
| Public route table: 0.0.0.0/0 → IGW |  Confirmed |
| Private route table: local only (no IGW) |  Confirmed |
| ALB security group: HTTP open to internet | Confirmed |
| EC2 security group: HTTP only from ALB SG |  Confirmed |
| ec2-web-1 running in private subnet | 3/3 status checks passed |
| ec2-web-2 running in private subnet |  3/3 status checks passed |
| Target group created with both instances healthy |  2/2. targets healthy |
| ALB active — internet-facing — HTTP:80 |  State: Active |
| Round-robin confirmed (browser alternates servers) | Confirmed |
| Direct private IP access blocked |  Connection timeout confirmed |
| All resources within AWS Free Tier |  t2.micro + ALB 750 hrs/mo |

---

## 12. Skills Demonstrated

### 12.1 AWS Services Used

| Service | Usage |
|---|---|
| **Amazon VPC** | Custom VPC, public/private subnets, IGW, route tables, SGs |
| **Amazon EC2** | t2.micro launch, AMI, user-data script, status checks |
| **Elastic Load Balancing** | Application LB, target groups, listener, health checks |
| **IAM / Security** | Least-privilege SG rules — EC2 only via ALB |

### 12.2 Networking Concepts Applied

- CIDR block planning for VPC and subnet segmentation
- Public vs private subnet design — separation of concerns
- IGW attachment and default route configuration
- Inbound/outbound security group chaining (alb-sg → ec2-sg)
- Layer 7 load balancing with HTTP health checks and round-robin distribution




