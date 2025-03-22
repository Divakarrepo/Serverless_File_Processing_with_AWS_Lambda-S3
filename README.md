

# Serverless File Processing with AWS Lambda & S3

This guide will help you build a serverless file processing system using AWS Lambda, S3, and Python (Boto3).

## Project Overview
1. User uploads a file to an S3 bucket.
2. Lambda function is triggered upon file upload.
3. The function processes the file (e.g., format conversion, content extraction, or validation).
4. The processed file is saved in another S3 bucket.

## Step 1: Set Up AWS Services
### 1.1 Create Two S3 Buckets
- **Source Bucket** → `bucket-uploadfile-source`
- **Destination Bucket** → `bucket-processedfile-destination`

## Step 2: Create an IAM Role for Lambda
1. Go to AWS IAM → Roles → Create Role.
2. Select AWS Service → Lambda.
3. Attach the following policies:
   - `AmazonS3FullAccess` (for reading/writing to S3)
   - `CloudWatchLogsFullAccess` (for logging)
4. Name the role **LambdaS3FileProcessingRole** and create it.

## Step 3: Create AWS Lambda Function
1. Navigate to AWS Lambda → Create function.
2. Choose **Author from scratch**.
3. Set:
   - **Function Name**: `FileProcessingLambda`
   - **Runtime**: Python 3.x
   - **Execution Role**: Use `LambdaS3FileProcessingRole`.
4. Click **Create Function**.

## Step 4: Write the Lambda Function Code
We will process text files by converting them to uppercase and saving them in the output bucket.

### 4.1 Update `lambda_function.py`
```python
import boto3
import json

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Extract bucket and file details from event
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']
    
    destination_bucket = "bucket-processedfile-destination"
    processed_key = f"processed-{object_key}"

    # Download file from S3
    response = s3.get_object(Bucket=source_bucket, Key=object_key)
    file_content = response['Body'].read().decode('utf-8')

    # Process the file (convert text to uppercase)
    processed_content = file_content.upper()

    # Upload processed file to destination S3 bucket
    s3.put_object(
        Bucket=destination_bucket,
        Key=processed_key,
        Body=processed_content,
        ContentType='text/plain'
    )

    return {
        'statusCode': 200,
        'body': json.dumps(f'File processed and saved to {destination_bucket}/{processed_key}')
    }
```

## Step 5: Configure S3 Event Trigger
1. Open **S3** → Source Bucket (`bucket-uploadfile-source`).
2. Go to **Properties** → **Event Notifications**.
3. Create a new event:
   - **Event Name**: `FileUploadTrigger`
   - **Event Type**: `Put`
   - **Lambda Function**: Select `FileProcessingLambda`.

## Step 6: Test the Lambda Function
1. Upload a text file (`sample.txt`) to `bucket-uploadfile-source`.
2. Check `bucket-processedfile-destination` for `processed-sample.txt`.
3. The content should be in uppercase.

## Step 7: Monitor in CloudWatch
1. Go to **CloudWatch Logs**.
2. Find `FileProcessingLambda` logs.
3. Check for errors or debugging info.
