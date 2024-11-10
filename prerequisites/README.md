# AWS Environment Prerequisites Setup

Let's set up your AWS environment using the AWS CLI and bash, referencing all placeholders as variables. We'll perform the following steps:

## **Step 1: Set Your AWS Account ID**

First, retrieve your AWS Account ID using the AWS CLI and set it as an environment variable.

```bash
export ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
echo "Your AWS Account ID is: $ACCOUNT_ID"
```

## **Step 2: Create an IAM Role with Lambda Execution Permissions**

### **Step 2.1: Define Variables**

Set variables for your role name and AWS region.

```bash
export ROLE_NAME=MyLambdaExecutionRole  # Replace with your desired role name
export AWS_REGION=us-east-1              # Replace with your desired AWS region
```

### **Step 2.2: Create a Trust Policy JSON File**

Create a trust policy that allows Lambda to assume the role.

```bash
cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

### **Step 2.3: Create the IAM Role**

Create the IAM role using the trust policy.

```bash
aws iam create-role \
    --role-name $ROLE_NAME \
    --assume-role-policy-document file://trust-policy.json
```

### **Step 2.4: Attach Policies to the Role**

Attach the **AWSLambdaBasicExecutionRole** policy to allow basic Lambda execution.

```bash
aws iam attach-role-policy \
    --role-name $ROLE_NAME \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

**Attach Additional Policies:**

If your Lambda function needs access to other AWS services, attach additional policies.

- **Amazon S3 Full Access:**

  ```bash
  aws iam attach-role-policy \
      --role-name $ROLE_NAME \
      --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
  ```

- **Amazon Bedrock Full Access:**

  As of my knowledge cutoff in September 2021, Amazon Bedrock may not have a predefined policy. You might need to create a custom policy or check if a managed policy is available.

**Example: Create a Custom Policy for Bedrock Access**

Create a policy document `bedrock-policy.json` with the necessary permissions.

```bash
cat > bedrock-policy.json << EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:*"
            ],
            "Resource": "*"
        }
    ]
}
EOF
```

Create the policy:

```bash
export POLICY_ARN=$(aws iam create-policy \
    --policy-name BedrockFullAccessPolicy \
    --policy-document file://bedrock-policy.json \
    --query 'Policy.Arn' --output text)
```

Attach the custom policy to your role:

```bash
aws iam attach-role-policy \
    --role-name $ROLE_NAME \
    --policy-arn $POLICY_ARN
```

### **Step 2.5: Retrieve the Role ARN**

Get the ARN of the IAM role and set it as an environment variable.

```bash
export LAMBDA_ROLE=$(aws iam get-role \
    --role-name $ROLE_NAME \
    --query 'Role.Arn' --output text)
echo "Lambda Role ARN: $LAMBDA_ROLE"
```

## **Step 3: Create an S3 Bucket**

### **Step 3.1: Define the S3 Bucket Name**

Set the S3 bucket name, ensuring it's globally unique. Incorporate your account ID to help with uniqueness.

```bash
export S3_BUCKET=my-bucket-$ACCOUNT_ID  # Replace 'my-bucket' with your preferred bucket prefix
echo "S3 Bucket Name: $S3_BUCKET"
```

### **Step 3.2: Create the S3 Bucket**

Create the S3 bucket in your specified region.

```bash
aws s3 mb s3://$S3_BUCKET --region $AWS_REGION
```

**Verify the Bucket Creation:**

```bash
aws s3 ls | grep $S3_BUCKET
```

## **Step 4: Export Environment Variables**

Verify it all necessary environment variables are set by running:

```bash
echo "ACCOUNT_ID: $ACCOUNT_ID"
echo "ROLE_NAME: $ROLE_NAME"
echo "LAMBDA_ROLE: $LAMBDA_ROLE"
echo "AWS_REGION: $AWS_REGION"
echo "S3_BUCKET: $S3_BUCKET"
```

---

## **Summary**

You have now:

- Set your AWS Account ID in `$ACCOUNT_ID`.
- Created an IAM role with the necessary permissions for Lambda, and stored its ARN in `$LAMBDA_ROLE`.
- Created a globally unique S3 bucket and stored its name in `$S3_BUCKET`.
- Exported all relevant environment variables for easy reference in your terminal session.

---

## **Next Steps**

With the prerequisites fulfilled, you can proceed to:

- **Package your Lambda function code** and upload it to the S3 bucket.
- **Create the Lambda function** using AWS CLI, referencing the S3 bucket and IAM role.
- **Configure and test your Lambda function** to ensure it interacts correctly with AWS Bedrock and any other services you’re using.

---

## **Additional Tips**

- **AWS CLI Configuration:**

  Ensure your AWS CLI is configured with the necessary credentials and default region.

  ```bash
  aws configure
  ```

- **Check Existing IAM Roles and Policies:**

  List your IAM roles:

  ```bash
  aws iam list-roles
  ```

  List your IAM policies:

  ```bash
  aws iam list-policies --scope Local
  ```

- **Cleaning Up:**

  Remember to clean up any resources you create to avoid unnecessary charges.

- **Persisting Environment Variables:**

  The exported environment variables are session-specific. If you open a new terminal session, you'll need to export them again or include them in your shell profile (`~/.bashrc` or `~/.bash_profile`).
