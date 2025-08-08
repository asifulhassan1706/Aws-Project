# Serverless Image Processing Pipeline with AWS 

This guide walks you through building a serverless image processing pipeline using AWS services: 

**S3, IAM ROLE, DynamoDB, Lambda and CloudFront.**

## Architecture Overview

1.	**Raw Images S3 Bucket** – Stores original uploaded images.
2.	**AWS Lambda** – Resizes and watermarks images.
3.    **IAM ROLE** -  Grants necessary permissions to access S3 and DynamoDB.
4.	**Processed Images S3 Bucket** – Stores processed images.
5.	**DynamoDB** – Stores metadata (filename, size, timestamp).
6.	**CloudFront** – Distributes processed images globally.

## Step-by-Step Setup

### Step 1. Create Source and Destination S3 Buckets:

1.	**Source bucket  -** Sample Name: ``image-source-bucket`` (**Private**)

2. **Destination bucket  -**  Sample Name: ``image-destination-bucket`` (**Private**)

3.	**Enable** Block all public access for both.

4.	Add appropriate **bucket policies** for **Lambda and CloudFront access**.

### Step 2. Create IAM Role for Lambda:

•	**Trusted entity:**  AWS Service → Lambda.

•	**Attach policies:**
```url
      AmazonS3FullAccess (read/write to both buckets)
      AmazonDynamoDBFullAccess (for metadata storage)
      CloudWatchLogsFullAccess` (for debugging)
```
• **Name it:** ``LambdaImageProcessingRole``
##### Why needed: Without this role, Lambda will fail to access S3, DynamoDB or logs.
________________________________________
### Step 3. Create DynamoDB Table:
**AWS Console → DynamoDB →** **Create table**

**Table name:** ``ImageMetadata``

**Partition key:** filename (String)

Leave other options `default` → **Create table**

________________________________________
### Step 4. Create Lambda Function:

**Sample Name:** `ImageProcessingFunction`

**Runtime:** `Python 3.12`

**Execution role:** `LambdaImageProcessingRole`

**Attach Pillow Layer** (Example ARN for us-east-1):

```url
   arn:aws:lambda:us-east-1:770693421928:layer:Klayers-p312-Pillow:6
```

**Lambda Code:**

```url
import boto3

from PIL import Image, ImageDraw

import os

from datetime import datetime

s3 = boto3.client('s3')

dynamodb = boto3.resource('dynamodb')

table = dynamodb.Table('ImageMetadata')

def lambda_handler(event, context):``

    try:

        bucket = event['Records'][0]['s3']['bucket']['name']

        key = event['Records'][0]['s3']['object']['key']

        download_path = f"/tmp/{os.path.basename(key)}"
        processed_key = f"processed-{os.path.basename(key)}"
        upload_path = f"/tmp/{processed_key}"

        s3.download_file(bucket, key, download_path)

        img = Image.open(download_path)
        img = img.resize((800, 600))
        draw = ImageDraw.Draw(img)
        draw.text((10, 10), "Watermark", fill=(255, 255, 255))
        img.save(upload_path)

        s3.upload_file(upload_path, 'processed-image-bucket', processed_key)

        table.put_item(Item={
            'filename': processed_key,
            'size': os.path.getsize(upload_path),
            'timestamp': datetime.utcnow().isoformat()
        })

        return {'statusCode': 200, 'body': f'Successfully processed {key}'}
    except Exception as e:
        return {'statusCode': 500, 'body': str(e)}
```

#### Test Event Example:
```url
{
  "Records": [
    {
      "s3": {
        "bucket": {"name": "raw-image-bucket"},
        "object": {"key": "test-image.jpg"}
      }
    }
  ]
}
```
### Step 5.  Set Environment Variables:

**SOURCE_BUCKET =** your source bucket name

**DEST_BUCKET =** your destination bucket name

### Step 6. Configure CloudFront:

**Origin:** ``processed-image-bucket.s3.amazonaws.com``

**Set Redirect HTTP to HTTPS**

**Cache policy:** CachingOptimized

**Default Root Object:** ``index.html``

**Upload an ``index.html`` file in the destination bucket for testing:**
```url
<!DOCTYPE html>
<html>
<body>
<h1>Processed Image</h1>```
<img src="processed-photo.jpg" alt="Processed Image">
</body>
</html>
```

### Step 7. Test the Pipeline:
1.	Upload ``photo.jpg`` → **raw bucket**.

2.	**Lambda processes it** → Upload ``processed-photo.jpg`` to processed bucket.

3.	**DynamoDB** entry created with metadata.

4.	View via **CloudFront URL**.

### Notes:

•	Make sure both buckets are **private**.

•	**Lambda must have correct permissions** for both **buckets** and **DynamoDB**.

•	**CloudFront** should point to the **destination bucket**.

```url
✅You now have a fully functional serverless image processing pipeline with automatic resizing, watermarking, metadata storage and global distribution.
```
