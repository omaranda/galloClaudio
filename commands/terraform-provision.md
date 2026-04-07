---
name: terraform-provision
description: Scaffold or extend Terraform infrastructure for AWS resources — ECS, RDS, Lambda, S3, SQS, VPC, IAM — following module-based, workspace-aware standards.
---

You are acting as the devops-aws agent. Provision AWS infrastructure with Terraform.

**Step 1 — Gather requirements (ask if not provided):**
- What AWS resources are needed? (ECS service, RDS, S3 bucket, SQS queue, Lambda, VPC, etc.)
- Which environment? (dev / staging / prod)
- Does a VPC already exist, or does one need to be created?
- Any specific sizing requirements? (instance type, storage, concurrency)
- Data sensitivity requirements? (encryption, VPC isolation, multi-AZ)

**Step 2 — Read existing Terraform code:**
Check for existing modules, `terraform.tfvars`, `backend.tf`, and `provider.tf` before writing new code.

**Step 3 — Generate Terraform code:**

**Module structure:**
```
infrastructure/
├── modules/
│   └── <resource-name>/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       └── terraform.tfvars
├── backend.tf
└── provider.tf
```

**Required in every resource:**
```hcl
tags = {
  Project     = var.project_name
  Environment = var.environment
  ManagedBy   = "terraform"
  Owner       = var.owner
}
```

**Backend config (S3 + DynamoDB locking):**
```hcl
terraform {
  backend "s3" {
    bucket         = "<project>-terraform-state"
    key            = "<service>/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

**Step 4 — Security standards:**
- [ ] No hardcoded account IDs or ARNs — use `data` sources
- [ ] IAM roles with least-privilege policies
- [ ] S3 buckets: `block_public_acls = true`, `versioning = enabled`, `server_side_encryption_configuration`
- [ ] RDS: `storage_encrypted = true`, `backup_retention_period >= 7`, `deletion_protection = true` for prod
- [ ] Security groups: default deny, only required ports open
- [ ] Secrets in AWS Secrets Manager — reference via `data.aws_secretsmanager_secret_version`

**Step 5 — Provide:**
- All `.tf` files (main, variables, outputs)
- Sample `terraform.tfvars` for the target environment
- Commands to initialize and apply:
  ```bash
  terraform init
  terraform workspace select dev
  terraform plan -var-file=environments/dev/terraform.tfvars
  terraform apply -var-file=environments/dev/terraform.tfvars
  ```
