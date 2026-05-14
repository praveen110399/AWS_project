# Automated S3-to-EC2 Web Deployment


> Automatically sync website files from an **Amazon S3 bucket** to an **EC2 instance** (`/var/www/html`) using a scheduled cron job — fully within the **AWS Free Tier**.

---

##  Architecture Overview

```
 ┌──────────────┐       aws s3 sync       ┌─────────────────────┐       HTTP
 │  S3 Bucket   │ ──────────────────────▶ │  EC2 t2.micro       │ ──────────▶  Browser
 │  (website    │    (every 5 minutes)    │  Apache /var/www/html│
 │   files)     │      via cron job       │  Amazon Linux 2023   │
 └──────────────┘                         └─────────────────────┘
```

---

## Table of Contents

- [Architecture Overview](#-architecture-overview)

- [AWS Free Tier Usage](#-aws-free-tier-usage)
- [Step 1 — Launch EC2 Instance](#step-1--launch-ec2-instance)
- [Step 2 — Create S3 Bucket & IAM Role](#step-2--create-s3-bucket--iam-role)
- [Step 3 — Install Apache & AWS CLI](#step-3--install-apache--aws-cli)
- [Step 4 — Create the Sync Script](#step-4--create-the-sync-script)
- [Step 5 — Configure Cron Job](#step-5--configure-cron-job)
- [Step 6 — Upload & Test](#step-6--upload--test)
- [Project Structure](#-project-structure)
- [Troubleshooting](#-troubleshooting)
- [License](#-license)

---

---

##  AWS Free Tier Usage

| Service | Free Tier Limit | Used In This Project |
|---|---|---|
| EC2 t2.micro | 750 hrs/month (12 months) | Web server |
| Amazon S3 storage | 5 GB | Website files |
| S3 GET requests | 20,000/month | Cron sync |
| S3 → EC2 transfer | Free (same region) | File sync |
| EBS Storage | 30 GB | EC2 root volume |

>  **Important:** Keep your S3 bucket and EC2 instance in the **same AWS region** to avoid data transfer charges.

---

## Step 1 — Launch EC2 Instance

### 1.1 Go to EC2 Console

Navigate to **AWS Console → EC2 → Launch Instance**

![EC2 Launch Screenshot](https://github.com/praveen110399/Aws-project/blob/348cfc3ca2dd5eb099e3e46440f81b62fe6cd8e2/s3-deploy/ec2.jpg)

### 1.2 Configure the Instance

| Setting | Value |
|---|---|
| **AMI** | Amazon Linux 2023 (or Ubuntu 22.04 LTS) |
| **Instance Type** | `t2.micro` (Free Tier eligible) |
| **Storage** | 8 GB gp3 EBS |
| **Security Group** | Allow **HTTP (port 80)** and **SSH (port 22)** |


---

## Step 2 — Create S3 Bucket & IAM Role

### 2.1 Create the S3 Bucket

![S3 bucket Screenshot](https://github.com/praveen110399/Aws-project/blob/348cfc3ca2dd5eb099e3e46440f81b62fe6cd8e2/s3-deploy/S3.jpg)

![s3_file](https://github.com/praveen110399/Aws-project/blob/348cfc3ca2dd5eb099e3e46440f81b62fe6cd8e2/s3-deploy/S2file.jpg) 

### 2.2 Create IAM Role (No Access Keys Needed)

Navigate to **IAM → Roles → Create Role → EC2** and attach the policy below:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::ec2-web-988826734722-us-east-1an",
        "arn:aws:s3:::ec2-web-988826734722-us-east-1an/*"
      ]
    }
  ]
}
```

![IAM Role Screenshot](https://github.com/praveen110399/Aws-project/blob/348cfc3ca2dd5eb099e3e46440f81b62fe6cd8e2/s3-deploy/IAM.jpg)

>  Using an IAM Role is safer than storing access keys on the EC2 instance.

---

## Step 3 — Install Apache & AWS CLI

SSH into your EC2 instance and run the following:

### Amazon Linux 2023

```bash
# Update packages
sudo dnf update -y

# Install Apache web server
sudo dnf install httpd -y

# Start and enable Apache on boot
sudo systemctl start httpd
sudo systemctl enable httpd

# Verify Apache is running
sudo systemctl status httpd

# AWS CLI v2 is pre-installed on Amazon Linux 2023
aws --version
```
### Test Apache is Serving

```bash
curl http://localhost
# Should return HTML content
```

---

## Step 4 — Create the Sync Script

Create the script at `/home/ec2-user/sync-site.sh`:

```bash
sudo nano /home/ec2-user/sync-site.sh
```

Paste the following content:

```bash
#!/bin/bash

BUCKET="s3://my-website-bucket"
DEST="/var/www/html"
LOG="/var/log/s3-sync.log"
REGION="ap-south-1"

echo "--- Sync started: $(date) ---" >> $LOG

/usr/local/bin/aws s3 sync $BUCKET $DEST \
  --delete \
  --region $REGION \
  --exact-timestamps >> $LOG 2>&1

echo "--- Sync completed: $(date) ---" >> $LOG
echo "" >> $LOG
```

Make the script executable:

```bash
chmod +x /home/ec2-user/sync-site.sh
```

Test the script manually:

```bash
sudo /home/ec2-user/sync-site.sh

# Check the log output
tail -20 /var/log/s3-sync.log
```

> **Important:** Always use the **full path** `/usr/local/bin/aws` inside the script. Cron runs with a minimal `PATH` and won't find `aws` by its short name.

| Flag | Purpose |
|---|---|
| `--delete` | Removes files from EC2 that were deleted in S3 |
| `--exact-timestamps` | Forces sync when timestamps differ, not just size |
| `--region` | Ensures same-region transfer (avoids charges) |

---

## Step 5 — Configure Cron Job

Open root's crontab (required to write to `/var/www/html`):

```bash
sudo crontab -e
```

Add the following line at the bottom:

```cron
*/5 * * * * /home/ec2-user/sync-site.sh
```

### Cron Expression Breakdown

```
┌───────────── minute (*/5 = every 5 minutes)
│ ┌─────────── hour   (* = every hour)
│ │ ┌───────── day    (* = every day)
│ │ │ ┌─────── month  (* = every month)
│ │ │ │ ┌───── weekday (* = every day)
│ │ │ │ │
*/5 * * * *  /home/ec2-user/sync-site.sh
```

**Common cron intervals:**

| Schedule | Expression |
|---|---|
| Every 5 minutes | `*/5 * * * *` |
| Every 15 minutes | `*/15 * * * *` |
| Every hour | `0 * * * *` |
| Daily at midnight | `0 0 * * *` |

### Verify Cron is Active

```bash
# List scheduled cron jobs
sudo crontab -l

# Watch the log file in real time
tail -f /var/log/s3-sync.log
```

---

## Step 6 — Upload & Test

Upload your website files to S3:

```bash
# Upload a single file
aws s3 cp index.html s3://my-website-bucket/

# Sync an entire local folder
aws s3 sync ./my-website/ s3://my-website-bucket/
```

Wait up to **5 minutes** (one cron cycle), then visit your EC2 public IP in a browser:

```
http://34.203.224.252
```

Your website should be live! 

![Result-1](https://github.com/praveen110399/Aws-project/blob/348cfc3ca2dd5eb099e3e46440f81b62fe6cd8e2/s3-deploy/result-1.jpg)


![Result-1](https://github.com/praveen110399/Aws-project/blob/348cfc3ca2dd5eb099e3e46440f81b62fe6cd8e2/s3-deploy/result-2.jpg)




---




