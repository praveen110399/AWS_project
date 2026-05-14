# Serverless REST API with AWS Lambda, API Gateway & S3

A production-ready serverless architecture that accepts JSON payloads via a REST API, processes them through AWS Lambda, and persists the data as `.json` files in Amazon S3.

---

## Architecture Overview

```
[ Client (Postman/cURL) ]
         │
         │  HTTP POST /submit
         ▼
[ Amazon API Gateway ]
         │
         │  Trigger
         ▼
[ AWS Lambda (Python 3.12) ]
         │
         │  PutObject
         ▼
[ Amazon S3 Bucket ]
```

---

##  Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Create the S3 Bucket](#step-1-create-the-amazon-s3-bucket)
- [Step 2: Create the IAM Role](#step-2-create-the-iam-role-for-lambda)
- [Step 3: Create the Lambda Function](#step-3-create-the-aws-lambda-function)
- [Step 4: Create the API Gateway](#step-4-create-the-amazon-api-gateway)
- [Step 5: Testing & Results](#step-5-testing-and-expected-results)
- [Clean-Up](#️-project-clean-up)

---

## Prerequisites

- An active **AWS Account**
- Basic familiarity with the **AWS Management Console**
- A REST client for testing: **Postman**, **Insomnia**, or **cURL**

---

## Step 1: Create the Amazon S3 Bucket

The S3 bucket will serve as the persistent data store for all incoming JSON payloads.

1. Open the **AWS Management Console** and navigate to **S3**.
2. Click **Create bucket**.
3. Configure the bucket:
   - **Bucket name:** Enter a globally unique name (e.g., `serverless-data-bucket-praveen`)
   - **AWS Region:** Choose your preferred region (e.g., `us-east-1`)
4. Leave all other settings as default.
5. Click **Create bucket**.


![s3 bucket](https://github.com/praveen110399/Aws-project/blob/6da6af1e24c24f401af77b4992896a713828183e/Lamdba/S3.jpg)


## Step 2: Create the IAM Role for Lambda

By default, Lambda cannot access S3. We create a dedicated IAM role to grant the necessary permissions.

1. Navigate to the **IAM** (Identity and Access Management) console.
2. Click **Roles** in the left sidebar → **Create role**.
3. Select:
   - **Trusted entity type:** AWS service
   - **Use case:** Lambda
4. Click **Next** and attach the following policies:
   - `AWSLambdaBasicExecutionRole` — enables CloudWatch logging
   -  `AmazonS3FullAccess` — grants S3 write access _(restrict to your specific bucket in production)_
5. Click **Next**, name the role `lambda-s3-integration-role`, then click **Create role**.




![IAM Role](https://github.com/praveen110399/Aws-project/blob/6da6af1e24c24f401af77b4992896a713828183e/Lamdba/IAM.jpg)



## Step 3: Create the AWS Lambda Function

This is the core backend logic that parses the incoming request and writes data to S3.

### 3.1 — Create the Function

1. Navigate to the **Lambda** console → **Create function**.
2. Select **Author from scratch**.
3. Configure the function:
   - **Function name:** `api-gateway-to-s3-handler`
   - **Runtime:** Python 3.12 (or the latest stable version)
   - **Permissions:** Expand _Change default execution role_ → _Use an existing role_ → select `lambda-s3-integration-role`
4. Click **Create function**.


! [Lambda Function](https://github.com/praveen110399/Aws-project/blob/6da6af1e24c24f401af77b4992896a713828183e/Lamdba/lambda.jpg)


### 3.2 — Add the Function Code

Replace the default code in `lambda_function.py` with the following:

```python
import json
import boto3
import os
from datetime import datetime

s3 = boto3.client('s3')
# Replace with your actual bucket name or use an environment variable
BUCKET_NAME = 'serverless-data-bucket-praveen'

def lambda_handler(event, context):
    try:
        # 1. Parse incoming JSON body from API Gateway
        body = json.loads(event.get('body', '{}'))

        if not body:
            return {
                'statusCode': 400,
                'headers': {'Content-Type': 'application/json'},
                'body': json.dumps({'error': 'Empty request body'})
            }

        # 2. Generate a unique file name using a timestamp
        timestamp = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
        file_name = f"records/data_{timestamp}.json"

        # 3. Upload the data payload directly to S3
        s3.put_object(
            Bucket=BUCKET_NAME,
            Key=file_name,
            Body=json.dumps(body, indent=4),
            ContentType='application/json'
        )

        return {
            'statusCode': 201,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({
                'message': 'Data successfully saved to S3!',
                'saved_as': file_name
            })
        }

    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({'error': str(e)})
        }
```

### 3.3 — Deploy

Click **Deploy** to save and publish your changes.


## Step 4: Create the Amazon API Gateway

We use an **HTTP API** for this setup — it's faster, cheaper, and ideal for straightforward Lambda integrations.

1. Navigate to **API Gateway** → **Create API**.
2. Under **HTTP API**, click **Build**.
3. **Add Integration:**
   - Integration type: **Lambda**
   - Lambda function: `api-gateway-to-s3-handler`
4. **API name:** `ServerlessIngestionAPI` → Click **Next**.
5. **Configure routes:**
   - Method: `POST`
   - Resource path: `/submit`
   - Target integration: your Lambda function
6. Click **Next**.
7. **Configure stages:** Leave as `$default` with **Auto-deploy** enabled → Click **Next** → **Create**.

![API Gateway Routes Configuration ](https://github.com/praveen110399/Aws-project/blob/6da6af1e24c24f401af77b4992896a713828183e/Lamdba/API_GW.jpg)


Copy the **Invoke URL** from the dashboard. It will look like:


https://tvwq8pauw1.execute-api.us-east-1.amazonaws.com/submit


## Step 5: Testing and Expected Results

### 5.1 — Send a Test Request

Use Reqbin to send a `POST` request:

| Field   | Value                                                            |
|---------|------------------------------------------------------------------|
| Method  | `POST`                                                           |
| URL     | `https://tvwq8pauw1.execute-api.us-east-1.amazonaws.com/submit`    |
| Header  | `Content-Type: application/json`                                 |

**Request Body:**

```json
{
    "sensor_id": "TL-094",
    "status": "Active",
    "metrics": {
        "temperature": 24.5,
        "humidity": 62
    }
}
```

### 5.2 — Expected API Response

A successful request returns a `201 Created` status:

```json
{
    "message": "Data successfully saved to S3!",
    "saved_as": "records/data_2026-05-14_19-58-00.json"
}


```


![Successful API Response in Postman ](https://github.com/praveen110399/Aws-project/blob/6da6af1e24c24f401af77b4992896a713828183e/Lamdba/API_test.jpg)


### 5.3 — Verify in Amazon S3

1. Navigate back to your S3 bucket: `serverless-data-bucket-praveen`
2. Open the `records/` folder that was auto-created.
3. Click the timestamped `.json` file to confirm the payload was stored correctly.


![S3 Bucket showing records ](https://github.com/praveen110399/Aws-project/blob/6da6af1e24c24f401af77b4992896a713828183e/Lamdba/S3_files.jpg)
```

---


