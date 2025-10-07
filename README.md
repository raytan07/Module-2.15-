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

```

## AWS Secrets Manager ARN for prod/cart-service/credentials
## 3. Using the secret name prod/cart-service/credentials, derive a sensible ARN as the specific resource for access.

## Derived Sensible ARN

The Amazon Resource Name (ARN) for the secret `prod/cart-service/credentials` can be derived using the standard AWS Secrets Manager format. Because the exact AWS region and account ID are required for a complete ARN, they are included as placeholders.

---

## Breakdown of the ARN

*   **`arn:aws:secretsmanager`**: The standard prefix for AWS Secrets Manager resources.
*   **`<Region>`**: The specific AWS region where the secret is stored (e.g., `us-east-1`).
*   **`<AccountId>`**: The 12-digit AWS account ID.
*   **`secret:`**: The resource type within the Secrets Manager service.
*   **`prod/cart-service/credentials`**: The user-defined name of the secret.
*   **`-??????`**: The six random characters automatically appended by Secrets Manager to make the ARN unique.

## How to use this ARN in an IAM policy

For a more flexible IAM policy, you can use a wildcard (`*`) to grant permissions to secrets under a specific path, such as `prod/cart-service`. This is useful for future-proofing your policy without needing to update it for every new secret created in that path.

### Example IAM policy statement

```json
{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue",
    "secretsmanager:DescribeSecret"
  ],
  "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/cart-service/*"
}

```

## ✅ Summary Table

| Component            | Purpose                                              |
| -------------------- | ---------------------------------------------------- |
| IAM Role             | Grants permissions to EC2 without static credentials |
| IAM Policy           | Specifies allowed actions and resources              |
| Secret ARN           | Identifies the exact secret in Secrets Manager       |
| EC2 Instance Profile | Associates the role with the running EC2 instance    |
| Permissions Needed   | `GetSecretValue`, `DescribeSecret`                   |
