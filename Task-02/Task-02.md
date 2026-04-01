# Serverless Image Processing Pipeline on AWS

## Objective

The objective of this project is to build a **scalable and automated image processing pipeline using AWS services**.

This includes:

* Secure image upload using Pre-Signed URLs
* Automated image processing using Lambda
* Storage management using S3
* Data protection using encryption and cross-region replication

---

## Architecture

```text
Client → Pre-Signed URL → S3 (Raw Bucket) → Lambda Trigger → Processed Images (S3) → CRR
```

![Architecture Diagram](./images/architecture.png)

---

## 1. S3 Setup

### Configuration Details

* Create `raw-uploads-bucket`

  * Enable Versioning
  * Enable SSE-S3 Encryption
  * Configure CORS

* Create `processed-images-bucket`

  * Enable SSE-KMS Encryption

* Enable Cross-Region Replication (CRR) to `eu-west-1`

* Configure Lifecycle Rules

  * Transition objects to cheaper storage tiers

### Explanation

This setup provides:

* Secure object storage
* Version control
* Disaster recovery
* Cost optimization

### Visual Verification

![S3 Setup](./images/s3-setup.png)

---

## 2. Lambda Processing Function

### Configuration Details

* Runtime: Python 3.11
* Memory: 512 MB
* Timeout: 30 seconds
* Trigger: S3 (`PUT Object`)

### Steps

1. Create Lambda function
2. Add S3 trigger (`raw-uploads-bucket`)
3. Configure memory and timeout

### Explanation

Automatically processes images whenever a new file is uploaded to the raw bucket.

### Visual Verification

![Lambda Function](./images/lambda-function.png)

---

## 3. Lambda Layer (Pillow Setup)

### Steps

```bash
mkdir python
pip install pillow -t python/
zip -r pillow-layer.zip python
```

* Upload as Lambda Layer
* Attach to Lambda function

### Explanation

This layer provides the **Pillow library** required for image processing.

### Visual Verification

![Lambda Layer](./images/lambda-layer.png)

---

## 4. IAM Permissions

### Required Permissions

* `s3:GetObject`
* `s3:PutObject`
* `AWSLambdaBasicExecutionRole`

### Explanation

These permissions allow Lambda to:

* Read uploaded images
* Process them
* Write processed images back to S3

### Visual Verification

![IAM Role](./images/iam-role.png)

---

## 5. Image Processing Logic

### Function Logic

* Resize image to `800x600`
* Add watermark
* Save to processed bucket

### Code

```python
import json
import boto3
from PIL import Image, ImageDraw
import io
import urllib.parse

s3 = boto3.client('s3')
DEST_BUCKET = "processed-images-bucket"

def lambda_handler(event, context):

    for record in event['Records']:

        source_bucket = record['s3']['bucket']['name']
        object_key = urllib.parse.unquote_plus(
            record['s3']['object']['key']
        )

        response = s3.get_object(
            Bucket=source_bucket,
            Key=object_key
        )

        image_content = response['Body'].read()
        image = Image.open(io.BytesIO(image_content))

        image = image.resize((800, 600))

        draw = ImageDraw.Draw(image)
        draw.text((10, 10), "Watermark", fill=(255, 255, 255))

        buffer = io.BytesIO()
        image.save(buffer, format="JPEG")
        buffer.seek(0)

        s3.put_object(
            Bucket=DEST_BUCKET,
            Key=object_key,
            Body=buffer,
            ContentType='image/jpeg'
        )

    return {
        'statusCode': 200,
        'body': json.dumps('Processing complete')
    }
```

### Explanation

This Lambda function automatically:

* Retrieves uploaded image
* Resizes it
* Applies watermark
* Uploads processed output

### Visual Verification

![Image Processing](./images/image-processing.png)

---

## 6. Pre-Signed URL Generator

### Configuration Details

* Lambda generates `PUT` URL
* Validity: `15 minutes (900 seconds)`

### Code

```python
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):

    bucket_name = "raw-uploads-bucket"
    object_key = "test.jpg"

    url = s3.generate_presigned_url(
        ClientMethod='put_object',
        Params={
            'Bucket': bucket_name,
            'Key': object_key
        },
        ExpiresIn=900
    )

    return {
        'statusCode': 200,
        'body': url
    }
```

### Explanation

Generates a temporary secure upload URL without exposing AWS credentials.

### Visual Verification

![Pre Signed URL](./images/presigned-url.png)

---

## 7. Testing

### Upload using curl

```bash
curl -X PUT -T test.jpg "PASTE_URL_HERE"
```

### Expected Result

* Image uploaded to raw bucket
* Lambda triggered automatically
* Processed image stored in processed bucket

### Visual Verification

![Testing Output](./images/testing.png)

---

## 8. Storage Validation

### Validation Tasks

* Verify lifecycle transitions using AWS CLI / S3 Console
* Check CRR replication in `eu-west-1`

### Explanation

Confirms:

* Storage optimization
* Replication
* Disaster recovery readiness

### Visual Verification

![Storage Validation](./images/storage-validation.png)

---

## Key Learnings

* Pre-Signed URLs
* S3 CORS & Lifecycle Rules
* Lambda triggers
* Image processing using Pillow
* Cross-Region Replication (CRR)
* Encryption (`SSE-S3` & `SSE-KMS`)

---

## Final Outcome

This solution provides:

* Secure file upload mechanism
* Automated serverless image processing
* Cost-efficient storage lifecycle
* Disaster recovery using CRR

---
