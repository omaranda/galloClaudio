---
name: iam-review
description: Review or generate AWS IAM policies and roles, applying least-privilege principles for ECS tasks, Lambda functions, and CI/CD pipelines.
---

You are acting as the devops-aws agent. Review or generate AWS IAM policies.

**Step 1 — Understand the scope:**
- What service needs IAM permissions? (ECS task, Lambda, GitHub Actions, EC2 instance, developer)
- What AWS resources does it access? (S3 buckets, SQS queues, RDS, Secrets Manager, ECR, etc.)
- Read vs. write vs. admin access needed per resource?

**Step 2 — Read existing IAM config:**
Check for existing Terraform IAM resources, policy JSON files, or CDK IAM definitions.

**Step 3 — Apply least-privilege principles:**

**Forbidden in service policies:**
- `"Resource": "*"` with write/delete actions — always scope to specific ARNs
- `iam:*` permissions — no service should manage IAM
- `s3:DeleteBucket` or `rds:DeleteDBInstance` in application policies
- Wildcard `s3:*` on production data buckets

**S3 access pattern:**
```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::<project>-data-<env>/*"
}
```

**SQS consumer pattern:**
```json
{
  "Effect": "Allow",
  "Action": [
    "sqs:ReceiveMessage",
    "sqs:DeleteMessage",
    "sqs:GetQueueAttributes",
    "sqs:ChangeMessageVisibility"
  ],
  "Resource": "arn:aws:sqs:<region>:<account>:<queue-name>"
}
```

**Secrets Manager read-only:**
```json
{
  "Effect": "Allow",
  "Action": ["secretsmanager:GetSecretValue"],
  "Resource": "arn:aws:secretsmanager:<region>:<account>:secret:<project>/<env>/*"
}
```

**Step 4 — GitHub Actions OIDC (no long-lived keys):**
```json
{
  "Effect": "Allow",
  "Principal": { "Federated": "arn:aws:iam::<account>:oidc-provider/token.actions.githubusercontent.com" },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
      "token.actions.githubusercontent.com:sub": "repo:<org>/<repo>:ref:refs/heads/main"
    }
  }
}
```

**Step 5 — Output:**
- Complete IAM policy JSON
- Terraform `aws_iam_role` + `aws_iam_role_policy` resources
- Summary of what each statement allows and why
- Any over-privileged patterns found with recommended fix
