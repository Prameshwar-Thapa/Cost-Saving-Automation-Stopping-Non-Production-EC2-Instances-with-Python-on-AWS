✅ Final Project Goals
Feature	Details
EC2 Target	                Instances tagged with Environment=Non-Prod
Stop Schedule	            Every Friday 6:00 PM (Sydney Time)
Start Schedule	            Every Monday 8:00 AM (Sydney Time)
Notifications	            Send SNS email when instances are stopped or started
Security	                IAM role with ec2:DescribeInstances, ec2:StartInstances, ec2:                        StopInstances only for tagged resources
![Uploading Untitled Diagram.drawio.png…]()
Language	Python 3.10 with Boto3



📁 GitHub Directory Structure

ec2-scheduler-project/
│
├── lambda/
│   ├── stop_nonprod_instances.py
│   └── start_nonprod_instances.py
│
├── iam/
│   └── lambda_execution_role.json
│
├── sns/
│   └── setup_sns_notification.md
│
├── cloudwatch/
│   └── eventbridge_schedules.md
│
├── README.md
└── LICENSE
🔧 Step-by-Step Setup in AWS Console
🔹 Step 1: Tag EC2 Instances
In EC2 → Instances:

Add a tag to each non-production instance:

Key: Environment

Value: Non-Prod

🔹 Step 2: Create SNS Topic for Notifications
Go to SNS > Topics > Create topic

Choose:

Type: Standard

Name: ec2-scheduler-alerts

Create a subscription:

Protocol: Email

Endpoint: your-email@example.com

Confirm the subscription via the email link

📁 Save the ARN for later (e.g., arn:aws:sns:ap-southeast-2:123456789012:ec2-scheduler-alerts)

🔹 Step 3: Create IAM Role for Lambda
Go to IAM > Roles > Create role

Use case: Lambda

Attach this inline policy:

📄 iam/lambda_execution_role.json:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2Control",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    },
    {
      "Sid": "SNSPublish",
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:ap-southeast-2:123456789012:ec2-scheduler-alerts"
    }
  ]
}
Name the role: LambdaEC2SchedulerRole

🔹 Step 4: Lambda Function – Stop Instances
📄 lambda/stop_nonprod_instances.py:


import boto3
import os

ec2 = boto3.client('ec2')
sns = boto3.client('sns')
SNS_TOPIC_ARN = os.environ['SNS_TOPIC_ARN']

def lambda_handler(event, context):
    filters = [
        {'Name': 'tag:Environment', 'Values': ['Non-Prod']},
        {'Name': 'instance-state-name', 'Values': ['running']}
    ]
    
    response = ec2.describe_instances(Filters=filters)
    instances = [
        instance['InstanceId']
        for reservation in response['Reservations']
        for instance in reservation['Instances']
    ]
    
    if instances:
        ec2.stop_instances(InstanceIds=instances)
        message = f"Stopping EC2 instances: {instances}"
    else:
        message = "No running Non-Prod EC2 instances to stop."

    sns.publish(TopicArn=SNS_TOPIC_ARN, Message=message)
    print(message)
Set environment variable:

SNS_TOPIC_ARN → your SNS ARN

🔹 Step 5: Lambda Function – Start Instances
📄 lambda/start_nonprod_instances.py:


import boto3
import os

ec2 = boto3.client('ec2')
sns = boto3.client('sns')
SNS_TOPIC_ARN = os.environ['SNS_TOPIC_ARN']

def lambda_handler(event, context):
    filters = [
        {'Name': 'tag:Environment', 'Values': ['Non-Prod']},
        {'Name': 'instance-state-name', 'Values': ['stopped']}
    ]
    
    response = ec2.describe_instances(Filters=filters)
    instances = [
        instance['InstanceId']
        for reservation in response['Reservations']
        for instance in reservation['Instances']
    ]
    
    if instances:
        ec2.start_instances(InstanceIds=instances)
        message = f"Starting EC2 instances: {instances}"
    else:
        message = "No stopped Non-Prod EC2 instances to start."

    sns.publish(TopicArn=SNS_TOPIC_ARN, Message=message)
    print(message)
🔹 Step 6: Create Lambda Functions
For each function:

Go to Lambda > Create function

Name:

StopNonProdInstances

StartNonProdInstances

Runtime: Python 3.10

Role: Use LambdaEC2SchedulerRole

Upload code

Add environment variable:

Key: SNS_TOPIC_ARN

Value: SNS topic ARN

🔹 Step 7: Create EventBridge (CloudWatch) Scheduler Rules
🛑 Stop Non-Prod Instances (Friday 6 PM)
Go to EventBridge Scheduler > Create schedule

Type: Cron-based

Cron:


cron(0 8 ? * MON *)   → Start (8 AM Monday)
cron(0 18 ? * FRI *)  → Stop (6 PM Friday)
Time zone: Australia/Sydney

Flexible time: Off

Target: Lambda

Schedule name: stop-nonprod-weekend

📄 README.md (for GitHub)
markdown
Copy
Edit
# EC2 Cost Optimization with Python & AWS Lambda

This project automates the shutdown of Non-Prod EC2 instances on weekends and restarts them Monday morning, using:

- Python (Boto3)
- AWS Lambda
- SNS Alerts
- IAM Roles
- EventBridge Scheduler

## Features
- 🚀 Stop Non-Prod EC2 every Friday 6 PM (Sydney)
- 🔁 Start Non-Prod EC2 every Monday 8 AM
- 📬 Email notification via SNS
- 🔐 IAM least-privilege setup
- 💸 Save up to 60% on EC2 runtime

## Tags Used
Ensure EC2 instances are tagged:
```bash
Environment=Non-Prod
Directory Structure
See the /lambda, /iam, /cloudwatch, and /sns folders for setup details.








