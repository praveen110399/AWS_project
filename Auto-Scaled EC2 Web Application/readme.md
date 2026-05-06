#  AWS Auto-Scaled EC2 Web Application

**Production-grade, highly available web app on AWS Free Tier** — EC2 · Classic/Application Load Balancer · Auto Scaling · CloudWatch · SNS

---
##  Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Setup](#-project-setup)
  - [1. Security Group](#1-security-group)
  - [2. EC2 Instances](#2-ec2-instances)
  - [3. Load Balancer](#3-load-balancer)
  - [4. Auto Scaling Group](#4-auto-scaling-group)
  - [5. Scaling Policy](#5-scaling-policy)
- [Verification & Testing](#-verification--testing)
  - [Load Balancer Traffic Distribution](#load-balancer-traffic-distribution)
  - [Stress Test — Auto Scaling Trigger](#stress-test--auto-scaling-trigger)
- [Results](#-results)
- [Skills Demonstrated](#-skills-demonstrated)
- [Next Steps](#-next-steps)

---
##  Overview

This project deploys a **horizontally scalable, fault-tolerant web application** on AWS using only Free Tier resources. It demonstrates real-world cloud architecture patterns including:

- Multi-AZ load balancing with automatic health checks
- CPU-based horizontal auto scaling (scale-out and scale-in)
- End-to-end verification with a live stress test

| Field | Details |
|---|---|
| **AWS Region** | `us-east-1` (N. Virginia) |
| **AWS Account** | `988267347222` |
| **Instance Type** | `t3.micro` (Free Tier eligible) |
| **AMI** | Amazon Linux 2023 (`ami-0eb38b817b93460ac`) |
| **LB DNS** | `testing-382763039.us-east-1.elb.amazonaws.com` |
| **Date** | May 6, 2026 |

---

##  Architecture

```
                        ┌─────────────────────────┐
                        │       Internet / Users   │
                        └────────────┬────────────┘
                                     │ HTTPS
                        ┌────────────▼────────────┐
                        │   Classic / App Load     │
                        │       Balancer           │
                        │  (Internet-facing, 4 AZ) │
                        └──────┬──────────┬────────┘
                               │          │ round-robin
               ┌───────────────▼──┐   ┌───▼────────────────┐
               │   EC2 Test-1     │   │   EC2 Test-2        │
               │   t3.micro       │   │   t3.micro          │
               │   us-east-1d     │   │   us-east-1d        │
               └──────────────────┘   └─────────────────────┘
                                    │
                          ┌────────────▼─────────────┐
                          │    Auto Scaling Group    │
                          │  Min: 0 · Des: 1 · Max: 1│
                          │  Target CPU: 50%         │
                          └────────────┬─────────────┘
                                       │ metrics
                          ┌────────────▼─────────────┐
                          │     Amazon CloudWatch    │
                          │   CPU alarms · policies  │
                          └────────────┬─────────────┘
                                       │ alarm actions
                          ┌────────────▼─────────────┐
                          │       Amazon SNS         │
                          │    Email / SMS alerts    │
                          └──────────────────────────┘
```

---

## Tech Stack

| Service | Purpose |
|---|---|
| **Amazon EC2** | Web server compute (t3.micro, Amazon Linux 2023) |
| **Classic Load Balancer** | Internet-facing traffic distribution across instances |
| **Application Load Balancer** | Layer-7 routing, health checks |
| **EC2 Auto Scaling** | Horizontal scaling via Target Tracking policy |
| **Amazon CloudWatch** | CPU utilization metrics and alarm triggers |
| **Amazon SNS** | Email/SMS notifications on scaling events |
| **Amazon VPC** | Default VPC, multi-AZ subnets, security groups |

---

## Project Setup

### 1. Security Group

Created a security group **`testing`** to control traffic for EC2 instances.

- **Inbound:** SSH (port 22) from admin IP
- **Outbound:** All traffic + Custom TCP port `8080` to `0.0.0.0/0`

![Creating Security Group](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/creating_SG.jpg)


---

### 2. EC2 Instances

Launched **two EC2 instances** (`Test-1` and `Test-2`) for the web tier:

```
AMI     : Amazon Linux 2023 (ami-0eb38b817b93460ac)
Type    : t3.micro
AZ      : us-east-1d
SG      : launch-wizard (with SSH + HTTP access)
```

![Creating EC2](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/creating_EC2.jpg)

After launch, both instances passed **3/3 status checks**:

![EC2 Instances Running](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/EC2_instance.jpg)


---

### 3. Load Balancer

#### Classic Load Balancer

Created **`testing`** CLB as internet-facing across 4 Availability Zones using the default VPC.

![Creating Classic LB](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/Creating_LB.jpg)

#### Application Load Balancer

Also configured an **Application Load Balancer** (`testing load balancer`) with IPv4, internet-facing scheme.

![Creating ALB](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/Creating_ALB.jpg)

**Final active state — 2/2 instances in service:**

![Loadbalancer Active](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/Loadbalancer.jpg)

---

### 4. Auto Scaling Group

Created ASG **`testing_auto`** using Launch Template `test1 | Version Default`:

![ASG Created](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/ALB_ASG_result.jpg)


**Activity log confirms instance launch:**

![ASG Activity](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/autoscaling%20created%20successfully.jpg)

```
2026-05-06 11:44 AM — Desired capacity changed from 0 → 1
2026-05-06 11:45 AM — Instance i-0790589ab926661c5 launched successfully
```

## 5. Amazon CloudWatch

Amazon CloudWatch provides real-time observability for EC2 instances and the Auto Scaling Group. It collects CPU utilization metrics, powers the custom dashboard, and drives alarm-based scaling and notification actions.

### 5.1 CloudWatch Metrics


![CloudWatch Metrics](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/creating%20cloud%20watch.jpg)


### 5.2 CloudWatch Dashboard

A custom dashboard named **`EC2`** was created to visualise CPU utilization for both instances over time.

- CPU spiked to **95.9%** on Test-1 at approximately 11:40 AM during stress test
- Both instance metrics overlaid on the same chart for easy comparison

![CloudWatch dashboard](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/cloud_watch_dashboard.jpg)

### 5.3 CloudWatch Alarm

| Field | Value |
|---|---|
| **Alarm name** | TEST-1 ec2 instance usage Hig |
| **Namespace** | AWS/EC2 |
| **Metric** | CPUUtilization |
| **Instance ID** | `i-068dc0cee5f49040d` (Test-1) |
| **Statistic** | Average |
| **Period** | 5 minutes |
| **Threshold type** | Static |
| **Condition** | Greater than **60%** |
| **Datapoints** | 1 out of 1 to trigger alarm |
| **Alarm action** | Notify SNS topic: `CPUalert` |

![CloudWatch alarm](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/creating%20alarms.jpg)

---

## 6. Amazon SNS — Notification Alerts

Amazon SNS was configured to deliver **real-time email alerts** when CloudWatch alarms transition state, completing the end-to-end observability pipeline:

```
High CPU → CloudWatch Alarm → SNS Notification → Email Delivery
```

![SNS Creating](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/creating%20SNS.jpg)

### 6.1 SNS Topic

| Field | Value |
|---|---|
| **Topic name** | CPUalert |
| **Topic ARN** | `arn:aws:sns:us-east-1:988267347222:CPUalert` |
| **Display name** | CPU |
| **Topic type** | Standard |
| **Topic owner** | 988267347222 |

![SNS Creating](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/creating%20SNS.jpg)


### 6.2 Email Subscription

| Field | Value |
|---|---|
| **Subscription ID** | `8a06554e-c94b-482f-a438-dc6d3bb1805a` |
| **Protocol** | EMAIL |
| **Endpoint** | apraveen1103@gmail.com |
| **Status** |  Confirmed |

![Email Subscription](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/SNS%20with%20email.jpg)


### 6.3 Live Alarm Email — End-to-End Proof

When the stress test drove CPUUtilization above 60% on Test-1, CloudWatch transitioned **OK → ALARM** and SNS delivered an email within seconds.

| Field | Value |
|---|---|
| **Alarm name** | TEST-1 ec2 instance usage Hig |
| **State change** | OK → ALARM |
| **Threshold** | CPUUtilization > 60.0 |
| **Datapoint** | 203.03% (06/05/26 07:08:00 UTC) |
| **Timestamp** | Wednesday 06 May, 2026 07:13:33 UTC |
| **AWS Account** | 988267347222 |
| **Alarm ARN** | `arn:aws:cloudwatch:us-east-1:988267347222:alarm:TEST-1 ec2 instance usage Hig` |
| **Delivered to** | apraveen1103@gmail.com — received at 12:43 PM |

![Email notification](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/email%20notification.jpg)
---

## 7. End-to-End Verification

### 7.1 Load Balancer Round-Robin

Accessing the ALB DNS in a browser and refreshing confirmed traffic distribution between:
![result-1] (https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/result_1.jpg)


![result-2] (https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/result_2.jpg)

### 7.2 Stress Test — Full Observability Chain

A CPU stress test was run on Test-1 to trigger the full observability chain:

```bash
sudo dnf install stress -y
stress --cpu $(nproc) --timeout 60s
```
![stress test](https://github.com/praveen110399/Aws-project/blob/bf83bfaf00db7503065b4e374de7d72a89c99571/demo/stress%20test%20for%20autoscale.jpg)

**Result:** stress dispatched 2 CPU hogs → CPU hit **95.9%** → CloudWatch detected the spike → alarm fired (OK → ALARM) → SNS delivered email alert → Auto Scaling initiated a scale-out event ✅

---

## 8. Project Results

| Milestone | Status |
|---|---|
| Web app accessible via ALB DNS |  HTTP 200 — confirmed |
| Load balancing across EC2 instances |  Round-robin between Test-1 and Test-2 |
| Auto Scaling policy active |  Target Tracking at 50% CPU |
| CloudWatch metrics graphed |  CPUUtilization for both instances |
| CloudWatch dashboard created |  EC2 dashboard — real-time CPU chart |
| CloudWatch alarm configured |  CPU > 60% threshold on Test-1 |
| SNS topic created & subscribed |  CPUalert topic — email confirmed |
| Alarm email delivered on CPU spike |  Email received at 12:43 PM on trigger |
| Stress test validated full pipeline |  95.9% CPU → alarm → SNS → email |
| All resources within Free Tier | t3.micro, CLB, basic CloudWatch metrics |

---






