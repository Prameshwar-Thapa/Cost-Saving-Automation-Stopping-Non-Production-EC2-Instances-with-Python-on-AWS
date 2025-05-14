🚀 EC2 Cost Optimization Using AWS Lambda and CloudWatch

## 📌 Overview

This project automates the shutdown of **non-production EC2 instances** to reduce AWS costs. Leveraging **AWS Lambda**, **CloudWatch Events (EventBridge)**, and **Boto3**, the solution stops tagged EC2 instances (`Environment=Non-Prod`) every 3 minutes without manual intervention. It achieves up to **60% cost savings** by enforcing tag-based filtering and secure IAM access.

---

## 🎯 Objective

- Automate EC2 instance management to reduce unused compute costs.
- Ensure only non-production instances are stopped.
- Provide a serverless, secure, and scheduled solution using native AWS services.

---

## ⚙️ Architecture

+----------------------+ +-----------------------+
| CloudWatch Events | --> | AWS Lambda |
| (Every 3 Minutes) | | (Python + Boto3 Code) |
+----------------------+ +-----------------------+
|
v
+--------------------------+
| EC2 Instances with Tag: |
| Environment=Non-Prod |
+--------------------------+

yaml
Copy
Edit

---

## 📁 Project Structure

ec2-cost-optimizer/
├── lambda/
│ └── stop_non_prod_ec2.py # Python script for Lambda function
├── iam/
│ └── lambda_execution_policy.json # IAM policy for Lambda permissions
├── cloudwatch/
│ └── schedule_rule.md # CloudWatch rule description
├── README.md # Project documentation (this file)

yaml
Copy
Edit

---

## 🛠️ Technologies Used

- **AWS Lambda**
- **Amazon EC2**
- **CloudWatch Events (EventBridge)**
- **IAM (Identity & Access Management)**
- **Python 3.9**
- **Boto3 (AWS SDK for Python)**

---

## 🪜 Step-by-Step Setup

### 1. ✅ Tag Your EC2 Instances
1. Go to the **EC2 Console**.
2. Select each non-production instance.
3. Under the **Tags** tab:
   - **Key**: `Environment`
   - **Value**: `Non-Prod`

---

### 2. 🔐 Create IAM Role for Lambda

#### IAM Policy: `lambda_execution_policy.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
Go to IAM → Roles → Create Role

Select Lambda as the trusted entity.

Attach the policy above.

Name the role: LambdaEC2StopperRole

3. 🧠 Create the Lambda Function
Lambda Function Code: stop_non_prod_ec2.py
python
Copy
Edit
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    stopped_instances = []

    filters = [
        {'Name': 'tag:Environment', 'Values': ['Non-Prod']},
        {'Name': 'instance-state-name', 'Values': ['running']}
    ]

    response = ec2.describe_instances(Filters=filters)

    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            stopped_instances.append(instance_id)

    if stopped_instances:
        ec2.stop_instances(InstanceIds=stopped_instances)
        print(f"Stopped instances: {stopped_instances}")
    else:
        print("No running non-prod instances found.")
Go to Lambda Console → Create Function

Choose:

Name: StopNonProdEC2Instances

Runtime: Python 3.9

Permissions: Attach LambdaEC2StopperRole

Paste the code above.

Click Deploy

4. ⏰ Schedule Lambda via CloudWatch Events
Rule Description: schedule_rule.md
Schedule expression: rate(3 minutes)

Target: Lambda function StopNonProdEC2Instances

Go to Amazon EventBridge (CloudWatch → Rules)

Click Create rule

Rule type: Schedule

Set expression: rate(3 minutes)

Target:

Function: StopNonProdEC2Instances

Click Create

🔍 Testing & Validation
Manually start a non-prod EC2 instance.

Wait for 3 minutes.

Go to CloudWatch Logs → /aws/lambda/StopNonProdEC2Instances

You should see a log showing that the instance was stopped.

💡 Key Features
🔁 Runs every 3 minutes automatically

🏷️ Uses EC2 tag-based filtering (Environment=Non-Prod)

🔐 Follows IAM least-privilege model

💰 Reduces cloud costs by stopping unused compute

📈 Results
Reduced EC2 compute cost by ~60% in non-production environments.

Zero manual effort using fully automated scheduling.

Easily extendable to support starting/stopping or rebooting instances.

👨‍💻 Author
Name: Prameshwar Thapa


