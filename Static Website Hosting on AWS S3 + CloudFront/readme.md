#  Static Website Hosting on AWS S3 + CloudFront

> **S3 · CloudFront · Static Website Hosting · Bucket Policy**  
> Deploy a fast, globally distributed static website using AWS S3 and CloudFront CDN.

---

##  Project Info

| Field | Details |
|---|---|
| **Project** | Static Website on S3 + CloudFront |

| **AWS Region** | us-east-1 (N. Virginia) |
| **Document Type** | Technical Project Documentation |

---

##  Table of Contents

1. [Project Overview](#1-project-overview)
2. [Step 1 — Create S3 Bucket](#2-step-1--create-s3-bucket)
3. [Step 2 — Create CloudFront Distribution](#3-step-2--create-cloudfront-distribution)
4. [Access Your Site](#4-access-your-site)

---

## 1. Project Overview

This project demonstrates how to host a **fully static website** on AWS using **S3** as the origin and **CloudFront** as the CDN layer for global, low-latency delivery with HTTPS support.

### Architecture Summary

| Component | Details |
|---|---|
| **Storage** | Amazon S3 — static file hosting with public read access |
| **CDN** | Amazon CloudFront — global edge distribution |
| **Protocol** | HTTP → HTTPS redirect via CloudFront viewer policy |
| **Entry Point** | `index.html` as both index and error document |
| **Region** | us-east-1 |

---

## 2. Step 1 — Create S3 Bucket

### 2.1 Bucket Setup

1. Go to **AWS Console → S3 → Create Bucket**
2. Choose a **unique bucket name** (e.g. `my-portfolio-site-202`)
3. Select region: `us-east-1`
4. **Uncheck** "Block all public access" → confirm
5. Enable **Static Website Hosting**:
   - Index document: `index.html`
   - Error document: `index.html`


   

### 2.2 Bucket Policy

Add the following **Bucket Policy** to allow public read access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```
**S3 bucket **

![](https://github.com/praveen110399/Aws-project/blob/main/demo/S3_bucket.png?raw=true)

**static hosting enabled**

![static hosting enabled](https://github.com/praveen110399/Aws-project/blob/main/demo/Static_hosting.png?raw=true)

**Bucket policy**

![Bucket policy](https://github.com/praveen110399/Aws-project/blob/main/demo/bucket_policy.png?raw=true)

**Static website  
  
**![Static website](https://github.com/praveen110399/Aws-project/blob/main/demo/Static_website.png?raw=true)


---

## 3. Step 2 — Create CloudFront Distribution

1. Go to **AWS Console → CloudFront → Create Distribution**
2. **Origin domain:** Select your S3 bucket website endpoint
3. **Viewer protocol policy:** Redirect HTTP to HTTPS
4. **Default root object:** `index.html`
5. Click **Create Distribution** — takes ~10 minutes to deploy

---
![Distribution](https://github.com/praveen110399/Aws-project/blob/main/demo/cloud_front.png?raw=true)**  

## 4. Access Your Site

Once the CloudFront distribution is deployed, access your site via the CloudFront domain name:

```
https://<your-distribution-id>.cloudfront.net
```
![Access Your Site](https://github.com/praveen110399/Aws-project/blob/main/demo/Static_hosting_cloud_front.png?raw=true)
---

## 5. Project Results

| Milestone | Status |
|---|---|
| S3 bucket created with unique name |  Configured |
| Public access unblocked | Confirmed |
| Static website hosting enabled |  index.html set as root |
| Bucket policy applied for public read |  `s3:GetObject` allowed |
| CloudFront distribution created |  Deployed (~10 min) |
| HTTP → HTTPS redirect configured | Viewer protocol policy set |
| Site accessible via CloudFront URL | Global CDN delivery active |

---

