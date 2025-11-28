# MonthlyInventoryReport-Automation-AWSLambda-Python
MonthlyInventoryReport-Automation using AWSLambda and Python Boto3

# 📦 AWS EC2 / EBS / Snapshot Monthly Inventory – Lambda Automation

Automated Monthly Inventory Reports for EC2 Instances, EBS Volumes, and Snapshots
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/26b96962-7256-493d-a245-5009e43379a7" />

## 📘 Overview

This project provides an **AWS Lambda function** that automatically generates **monthly inventory reports** for:

* **EC2 Instances**
* **EBS Volumes**
* **EC2 Snapshots (latest snapshot per volume for the current month)**

The Lambda function extracts metadata using `boto3`, generates CSV files in memory, and uploads timestamped reports to **Amazon S3**.

This automation helps with:

✔ Compliance & audit readiness
✔ Monthly infrastructure reports
✔ Capacity planning
✔ Backup & snapshot verification

---

## ⚙️ AWS Services Involved

| AWS Service                 | Purpose                                           |
| --------------------------- | ------------------------------------------------- |
| **Lambda**                  | Executes the Python inventory script              |
| **EC2**                     | Source of instance, volume, and snapshot metadata |
| **S3**                      | Stores generated CSV reports                      |
| **CloudWatch Logs**         | Captures logs and execution details               |
| **EventBridge (Scheduled)** | Triggers monthly execution                        |

---

## 📂 Project Structure

```
aws-monthly-inventory/
│
├── lambda/
│   ├── lambda_function.py        # Main Lambda code
│   ├── requirements.txt          # Optional if using layers
│   └── README.md                 # This README
│
├── infrastructure/ (optional)
│   ├── terraform/
│   └── cloudformation/
│
└── docs/
    └── architecture-diagram.png (optional)
```

---

## 🏗️ Architecture Diagram

```
                ┌────────────────────────────┐
                │    Amazon EventBridge       │
                │  (Monthly Scheduled Trigger)│
                └──────────────┬─────────────┘
                               │
                               ▼
                      ┌────────────────┐
                      │ AWS Lambda     │
                      │ Python + boto3 │
                      └───────┬────────┘
                ┌─────────────┼──────────────────────────┐
                │             │                          │
                ▼             ▼                          ▼
        ┌────────────┐ ┌─────────────┐         ┌─────────────────┐
        │ EC2 API     │ │ EBS API     │         │ Snapshot API     │
        └────┬────────┘ └─────┬──────┘         └────────┬────────┘
             │                 │                           │
             └─────────────┬──┴───────────────┬──────────┘
                           ▼                    ▼
                    Generate CSV Files in Memory
                           ▼
                ┌───────────────────────────┐
                │        Amazon S3          │
                │  EC2 / EBS / Snapshot CSV │
                └───────────────────────────┘
```

---

## 📊 Reports Generated

### ✅ **1. EC2 Instance Monthly Inventory Report**

Stored in bucket:
`monthlyinventory-report-ind/EC2-Instance-Monthly-Inventory report/`

Includes:

* Instance ID / Name / Type / State
* Private/Public/Elastic IP
* AZ, VPC, Subnet
* OS Type, KeyPair
* ENI Creation Date
* Environment Tag

---

### ✅ **2. EBS Volume Monthly Inventory Report**

Stored in bucket:
`monthlyinventory-report/EBS-Volume-Monthly-Inventory report/`

Includes:

* Volume ID, Size, Type
* IOPS, Throughput
* AZ
* Encryption + KMS Key
* Attached Instance Info

---

### ✅ **3. EC2 Snapshot Monthly Report**

Stored in bucket:
`monthlyinventory-report/Snapshot-Monthly-Report/`

Includes:

* Latest snapshot (per volume) created in the current month
* Snapshot ID, Size, State
* Description
* Storage type
* Encryption

---

## 🧠 How It Works (Detailed Flow)

### Step 1 — Fetch EC2 Instances

Uses `describe_instances()` → extracts instance metadata
→ builds CSV → uploads to S3.

### Step 2 — Fetch EBS Volumes

Uses `describe_volumes()` + EC2 lookup for attachment
→ builds CSV → uploads to S3.

### Step 3 — Fetch Current Month Snapshots

Uses `describe_snapshots(OwnerIds=['self'])`
→ Filters current month
→ Selects latest snapshot per volume
→ Builds CSV → uploads to S3.

Each section is isolated in its own `try/except` so failure in one does **not** affect others.

---

## 🔐 IAM Permissions Required

Attach this policy to the Lambda IAM Role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": [
        "arn:aws:s3:::monthlyinventory-report-ind/*",
        "arn:aws:s3:::monthlyinventory-report/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 🚀 Deploying the Lambda

### Option 1 — AWS Console

1. Create a new Lambda (Python 3.x).
2. Paste the `lambda_function.py` code.
3. Configure IAM role with the policy above.
4. Update S3 bucket names in code.
5. Deploy.
6. Test the function manually.

---

### Option 2 — Terraform (Recommended)

If you want, I can generate:

✔ Terraform for Lambda + IAM Role
✔ S3 buckets
✔ EventBridge scheduled trigger
✔ CloudWatch log retention

Just tell me **"Generate Terraform"**.

---

## ⏱️ Scheduling Monthly Execution (EventBridge)

Example cron:

```
cron(0 0 1 * ? *)   → Run on 1st of every month at 00:00 UTC
```

---

## 📝 Environment Variables (Recommended)

Move config out of code:

| Variable          | Purpose                     |
| ----------------- | --------------------------- |
| `EC2_BUCKET`      | Bucket for EC2 reports      |
| `EBS_BUCKET`      | Bucket for EBS reports      |
| `SNAPSHOT_BUCKET` | Bucket for Snapshot reports |

---

## 🛠️ Local Development

Install dependencies:

```
pip install boto3
```

Run locally:

```python
python lambda_function.py
```

(Requires AWS credentials via AWS CLI or environment variables.)

---

## 🧩 Future Enhancements

* Add **Athena + Glue Crawler** to query CSVs
* Add **SNS notification** after report upload
* Add **pagination** for very large AWS environments
* Convert CSV → JSON / Parquet
* Add **organization-wide** reports using AWS Organizations


