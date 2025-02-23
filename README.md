# AWS Lambda-SQS-S3 Workshop

This workshop guides you through creating a serverless data processing pipeline using AWS Lambda, SQS, and S3.

## Architecture Overview
Lambda function reads messages from SQS queue and saves them to S3 bucket.

## Prerequisites
- AWS Account access (through IAM user)
- Basic Python knowledge
- Basic AWS services understanding

## Setup Steps

### 1. SQS Queue Creation
1. Go to SQS Console
2. Click "Create queue"
3. Name it `{your-name}-orders-queue`
4. Keep default settings (Standard queue)
5. Create queue

### 2. S3 Bucket Creation
1. Go to S3 Console
2. Click "Create bucket"
3. Name it `{your-name}-orders-bucket`
4. Keep default settings
5. Enable versioning (recommended)
6. Create bucket

### 3. Lambda Function Creation
1. Go to Lambda Console
2. Click "Create function"
3. Select "Author from scratch"
4. Function name: `{your-name}-order-processor`
5. Runtime: Python 3.12
6. Create function

### 4. Add SQS Trigger to Lambda
1. In Lambda function page
2. Click "Add trigger"
3. Select "SQS"
4. Select your queue
5. Keep batch size as 1
6. Click "Add"

## Code Snippets

### Lambda Function Code
```python
import json
import logging
import boto3
from datetime import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Initialize S3 client
s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Replace with your bucket name
    bucket_name = 'your-name-orders-bucket'
    
    logger.info(f"Received event: {json.dumps(event)}")
    
    for record in event['Records']:
        try:
            # Parse the message body
            message_body = json.loads(record['body'])
            
            # Create a unique file name using timestamp and order ID
            timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
            order_id = message_body['orderId']
            file_name = f"orders/{timestamp}-order-{order_id}.json"
            
            # Save to S3
            s3.put_object(
                Bucket=bucket_name,
                Key=file_name,
                Body=json.dumps(message_body, indent=2),
                ContentType='application/json'
            )
            
            logger.info(f"Saved order {order_id} to S3: {file_name}")
            
            # Process order details
            customer = message_body['customerName']
            total = message_body['totalAmount']
            logger.info(f"Processed order {order_id} from {customer} for ${total}")
            
        except Exception as e:
            logger.error(f"Error processing message: {e}")
    
    return {
        'statusCode': 200,
        'body': json.dumps('Messages processed and saved to S3')
    }
