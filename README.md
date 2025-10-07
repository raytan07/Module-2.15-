# Authorizing EC2 to Retrieve Secrets from AWS Secrets Manager

## 1. What is needed to authorize your EC2 to retrieve secrets?

To allow an **EC2 instance** to retrieve secrets from **AWS Secrets Manager**, you need the following:

1. **An IAM Role** with the appropriate permissions to read the secret.  
2. **An Instance Profile** attached to the EC2 instance. This allows the EC2 instance to assume the role automatically.  
3. **IAM Policy** that grants `secretsmanager:GetSecretValue` and optionally `secretsmanager:DescribeSecret` to the **specific secret**.  
4. The application running on the EC2 instance uses **temporary credentials** from the Instance Metadata Service (IMDS), so no hard-coding of credentials is required.

---

## 2. Derive the IAM Policy (JSON)

**Explanation**:

-secretsmanager:GetSecretValue → allows the application to retrieve the actual secret value.

-secretsmanager:DescribeSecret → allows describing metadata about the secret.

-Resource → should be the exact ARN of the secret. You can find this in the AWS Secrets Manager console or construct it manually.


Below is an example IAM policy that gives **read-only access to one specific secret**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadSpecificSecret",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret"
      ],
      "Resource": "arn:aws:secretsmanager:ap-southeast-1:123456789012:secret:prod/cart-service/credentials-AbCdEf"
    }
  ]
}

