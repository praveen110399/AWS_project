**Project 1: Static Website on S3 + CloudFront **

**Short Description: Hosting a Static Website on AWS**
This guide outlines the process of deploying a static website—consisting only of HTML, CSS, and JavaScript—using Amazon S3 and CloudFront without the need for a traditional server.  

Steps to create:-
==============================================
**Step 1 — Create S3 Bucket**

1\. Go to **\*\*AWS Console → S3 → Create Bucket\*\***

2\. Choose a unique bucket name (e.g. \`my-portfolio-site-202\`)

3\. Select a region (e.g. \`us-east-1\`)

4\. Uncheck **\*\*"Block all public access"\*\*** → confirm

5\. Enable **\*\*Static website hosting\*\***:

   - Index document: \`index.html\`

   - Error document: \`index.html\`

6\. Add this **\*\*Bucket Policy\*\*** (

\`\`\`json

{

  "Version": "2012-10-17",

  "Statement": \[

    {

      "Sid": "PublicReadGetObject",

      "Effect": "Allow",

      "Principal": "\*",

      "Action": "s3:GetObject",

      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/\*"

    }

  \]

}

**S3 bucket **

![](https://github.com/praveen110399/Aws-project/blob/main/demo/S3_bucket.png?raw=true)

**static hosting enabled**

![static hosting enabled](https://github.com/praveen110399/Aws-project/blob/main/demo/Static_hosting.png?raw=true)

**Bucket policy**

![Bucket policy](https://github.com/praveen110399/Aws-project/blob/main/demo/bucket_policy.png?raw=true)

**Static website  
  
**![Static website](https://github.com/praveen110399/Aws-project/blob/main/demo/Static_website.png?raw=true)

**Cloudfront:  
  
Step 2 — Create CloudFront Distribution**

**1. Go to \*\*AWS Console → CloudFront → Create Distribution\*\***

**2. Origin domain: Select your S3 bucket website endpoint**

**3. Viewer protocol policy: \*\*Redirect HTTP to HTTPS\*\***

**4. Default root object: \`index.html\`**

**5. Click \*\*Create Distribution\*\* — takes \~10 minutes to deploy**

![Distribution](https://github.com/praveen110399/Aws-project/blob/main/demo/cloud_front.png?raw=true)**  
**

**Access Your Site**

![Access Your Site](https://github.com/praveen110399/Aws-project/blob/main/demo/Static_hosting_cloud_front.png?raw=true)
