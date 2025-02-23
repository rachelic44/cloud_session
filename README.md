AWS Lambda-SQS-S3 Workshop
This workshop guides you through creating a serverless data processing pipeline using AWS Lambda, SQS, and S3.
Architecture Overview
Lambda function reads messages from SQS queue and saves them to S3 bucket.
Prerequisites

AWS Account access (through IAM user)
Basic Python knowledge
Basic AWS services understanding

Setup Steps
1. SQS Queue Creation

Go to SQS Console
Click "Create queue"
Name it {your-name}-orders-queue
Keep default settings (Standard queue)
Create queue

2. S3 Bucket Creation

Go to S3 Console
Click "Create bucket"
Name it {your-name}-orders-bucket
Keep default settings
Enable versioning (recommended)
Create bucket

3. Lambda Function Creation

Go to Lambda Console
Click "Create function"
Select "Author from scratch"
Function name: {your-name}-order-processor
Runtime: Python 3.12
Create function

4. Add SQS Trigger to Lambda

In Lambda function page
Click "Add trigger"
Select "SQS"
Select your queue
Keep batch size as 1
Click "Add"

Code Snippets
Lambda Function Code
pythonCopyimport json
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
SQS Test Message
jsonCopy{
    "orderId": "12345",
    "customerName": "John Doe",
    "items": [
        {
            "productId": "PRD-001",
            "quantity": 2,
            "price": 29.99
        },
        {
            "productId": "PRD-002",
            "quantity": 1,
            "price": 49.99
        }
    ],
    "totalAmount": 109.97,
    "orderDate": "2024-02-23T14:30:00Z"
}
Lambda Test Event
jsonCopy{
  "Records": [
    {
      "messageId": "19dd0b57-b21e-4ac1-bd88-01bbb068cb78",
      "receiptHandle": "MessageReceiptHandle",
      "body": "{\"orderId\": \"12345\", \"customerName\": \"John Doe\", \"items\": [{\"productId\": \"PRD-001\", \"quantity\": 2, \"price\": 29.99}, {\"productId\": \"PRD-002\", \"quantity\": 1, \"price\": 49.99}], \"totalAmount\": 109.97, \"orderDate\": \"2024-02-23T14:30:00Z\"}",
      "attributes": {
        "ApproximateReceiveCount": "1",
        "SentTimestamp": "1523232000000",
        "SenderId": "123456789012",
        "ApproximateFirstReceiveTimestamp": "1523232000001"
      },
      "messageAttributes": {},
      "md5OfBody": "7b270e59b47ff90a553787216d55d91d",
      "eventSource": "aws:sqs",
      "eventSourceARN": "arn:aws:sqs:region:123456789012:MyQueue",
      "awsRegion": "us-east-1"
    }
  ]
}
Required Permissions
IAM Group Permissions
Your IAM group should have these policies:

AWSLambda_FullAccess
AmazonSQSFullAccess
AWSLambdaBasicExecutionRole
CloudWatchLogsFullAccess
AmazonS3FullAccess
IAMFullAccess

Lambda Execution Role Permissions
Lambda function needs:

AWSLambdaBasicExecutionRole (for CloudWatch logs)
AWSLambdaSQSQueueExecutionRole (for SQS access)
AmazonS3FullAccess (for S3 access)

Testing & Verification

Test Lambda Function:

Use the Lambda test event provided above
Click "Test" button
Check execution results


Send Message via SQS:

Go to your SQS queue
Click "Send and receive messages"
Paste the SQS test message
Click "Send"


Check Results:

CloudWatch Logs: Look for successful processing messages
S3 Bucket: Check for new JSON files in orders/ folder



Common Issues & Solutions
Permission Issues

"Access denied to iam:CreateRole"

Solution: Add IAMFullAccess to your group permissions


"The function execution role does not have permissions to call ReceiveMessage on SQS"

Solution: Add AWSLambdaSQSQueueExecutionRole to Lambda execution role


"Access denied to s3:PutObject"

Solution: Add S3 permissions to Lambda execution role



How to Check Logs

In Lambda console:

Click "Monitor" tab
Click "View CloudWatch logs"
Check latest log stream


In CloudWatch:

Go to Log groups
Find /aws/lambda/{your-lambda-name}
Click on latest stream



Best Practices

Use unique names for all resources
Always check CloudWatch logs for errors
Remember to click "Deploy" after code changes
Test with small data first
Monitor AWS costs

Additional Resources

AWS Lambda Developer Guide
Amazon SQS Developer Guide
Amazon S3 Developer Guide
