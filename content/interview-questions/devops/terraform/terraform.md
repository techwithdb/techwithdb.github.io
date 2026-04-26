---
title: "Terraform Interview Questions & Answers (2026) Part 01"
description: "50+ Terraform interview questions and answers covering state management, modules, lifecycle, drift detection, and advanced scenarios — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["terraform", "iac", "interview", "devops", "aws"]
tool: "terraform"
level: "All Levels"
question_count: 50
draft: "false"
---

<div class="qa-list">

{{< qa num="1" q="What is Terraform and how does it work?" level="basic" >}}
**Ans:**

**Terraform** is an open-source Infrastructure as Code (IaC) tool by HashiCorp that lets you define and provision infrastructure using declarative configuration files (`.tf` files).

**How it works:**

```
Write .tf files → terraform init → terraform plan → terraform apply → infrastructure created
```

1. You write HCL (HashiCorp Configuration Language) describing desired infrastructure
2. `terraform init` downloads provider plugins
3. `terraform plan` shows what will be created/changed/destroyed
4. `terraform apply` makes the changes in real infrastructure
5. Terraform records state in `terraform.tfstate`

```hcl
# Example: Create an EC2 instance
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = {
    Name = "MyWebServer"
  }
}
```

```bash
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```
{{< /qa >}}

{{< qa num="2" q="What is the Terraform state file? Why is it used?" level="basic" >}}
**Ans:**

The **state file** (`terraform.tfstate`) is a JSON file that records the current state of infrastructure managed by Terraform. It maps your configuration to real-world resources.

**Why it's needed:**
- Tracks resource IDs created by providers (e.g., EC2 instance ID)
- Enables `terraform plan` to detect differences (desired vs actual)
- Manages dependency order between resources
- Stores metadata that providers don't expose via API

```json
// terraform.tfstate (simplified)
{
  "version": 4,
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "instances": [
        {
          "attributes": {
            "id": "i-1234567890abcdef0",
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t3.micro"
          }
        }
      ]
    }
  ]
}
```

```bash
# View current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web
```
{{< /qa >}}

{{< qa num="3" q="What is a Terraform remote backend and why is it important?" level="intermediate" >}}
**Ans:**

A **remote backend** stores the Terraform state file remotely (S3, Terraform Cloud, Azure Blob, GCS) instead of locally.

**Why it matters:**
- **Collaboration**: Multiple team members share the same state
- **State locking**: Prevents concurrent `apply` from corrupting state
- **Security**: State contains secrets — remote backends support encryption
- **Recovery**: State not lost if your laptop dies

```hcl
# S3 + DynamoDB backend (most common for AWS)
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true                          # Enable server-side encryption
    dynamodb_table = "terraform-state-lock"        # For state locking
  }
}
```

```bash
# After adding backend config:
terraform init   # Migrates local state to remote
```

**DynamoDB table for locking:**
```bash
aws dynamodb create-table \
  --table-name terraform-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```
{{< /qa >}}

{{< qa num="4" q="How does Terraform state locking work with S3 and DynamoDB?" level="intermediate" >}}
**Ans:**

```
terraform apply starts
        │
        ▼
Terraform writes lock record to DynamoDB:
  {
    "LockID": "my-bucket/prod/terraform.tfstate",
    "Operation": "OperationTypeApply",
    "Who": "user@host",
    "Created": "2024-01-01T10:00:00Z"
  }
        │
        ▼
Apply runs and modifies infrastructure
        │
        ▼
State written to S3 (atomic PUT operation)
        │
        ▼
Lock removed from DynamoDB
```

**What happens if two engineers run `apply` simultaneously:**
- First engineer acquires the lock in DynamoDB
- Second engineer's `apply` fails with: `Error acquiring the state lock`
- Second engineer must wait or manually force-unlock

```bash
# View current lock
aws dynamodb get-item \
  --table-name terraform-state-lock \
  --key '{"LockID": {"S": "my-bucket/prod/terraform.tfstate"}}'

# Force unlock (use with caution!)
terraform force-unlock <LOCK_ID>
```

**What if lock is lost mid-apply?**
- DynamoDB lock remains → next `apply` will fail
- Investigate whether apply completed partially
- Then `terraform force-unlock`
{{< /qa >}}

{{< qa num="5" q="What is Terraform drift? How do you detect and prevent it?" level="intermediate" >}}
**Ans:**

**Drift** occurs when real infrastructure differs from what Terraform's state file records — usually caused by manual changes in the cloud console or CLI.

**Detection:**
```bash
# terraform plan shows drift automatically
terraform plan
# If someone manually changed EC2 size to t3.large but TF config says t3.micro:
# ~ aws_instance.web
#   ~ instance_type = "t3.large" -> "t3.micro"

# terraform refresh updates state from real infra
terraform refresh    # Deprecated; now built into plan

# Explicit drift check
terraform plan -refresh-only   # Shows what changed outside Terraform
```

**Prevention strategies:**
```bash
# 1. Deny manual console access via IAM (SCPs in AWS Organizations)
# 2. Use AWS Config rules to detect manual changes
# 3. Use CloudTrail to audit all API calls
# 4. Run terraform plan in CI/CD on a schedule (drift detection pipeline)
# 5. Lock down console permissions - only Terraform role can modify infra
```

**Reconcile drift without downtime:**
```bash
# Option A: Import the manual change into state
terraform import aws_instance.web i-manually-changed-id

# Option B: Accept the drift (update TF code to match reality)
# Edit .tf file to reflect current state, then plan shows no changes

# Option C: Revert — apply to force infrastructure back to TF config
terraform apply  # Rewrites manual changes
```
{{< /qa >}}

{{< qa num="6" q="What is the difference between count and for_each? Why can count be dangerous?" level="intermediate" >}}
**Ans:**

| Feature | count | for_each |
|---------|-------|----------|
| Input type | Integer | Map or Set of strings |
| Resource address | `aws_instance.web[0]`, `[1]` | `aws_instance.web["prod"]`, `["staging"]` |
| Deletion risk | **HIGH** — reindexes on removal | Low — uses key |
| Identify by | Index number | Map key |

**Why `count` is dangerous:**

```hcl
# count example
variable "envs" {
  default = ["dev", "staging", "prod"]
}

resource "aws_instance" "web" {
  count = length(var.envs)
  ami   = "ami-xxx"
  tags  = { Name = var.envs[count.index] }
}

# State: web[0]=dev, web[1]=staging, web[2]=prod

# Now remove "staging" → list becomes ["dev", "prod"]
# State: web[0]=dev, web[1]=prod
# Terraform DESTROYS web[2] and RECREATES web[1]!
# → staging and prod are both recreated!
```

```hcl
# for_each is safer
resource "aws_instance" "web" {
  for_each      = toset(["dev", "staging", "prod"])
  ami           = "ami-xxx"
  tags          = { Name = each.key }
}
# State: web["dev"], web["staging"], web["prod"]
# Remove "staging" → only web["staging"] is destroyed. dev and prod untouched!
```
{{< /qa >}}

{{< qa num="7" q="What are lifecycle meta-arguments in Terraform? When would you use create_before_destroy or prevent_destroy?" level="intermediate" >}}
**Ans:**

`lifecycle` blocks control how Terraform handles resource creation, destruction, and updates.

```hcl
resource "aws_instance" "web" {
  # ...

  lifecycle {
    # Create new instance before destroying old one (zero-downtime replacement)
    create_before_destroy = true

    # Block destroy - Terraform will error if you try to destroy this
    prevent_destroy = true

    # Ignore changes to specific attributes (don't update if these change)
    ignore_changes = [tags, user_data]

    # Custom conditions (Terraform 1.2+)
    precondition {
      condition     = var.instance_type != "t1.micro"
      error_message = "t1.micro is not supported"
    }
  }
}
```

**When to use each:**

| Argument | Use case |
|----------|---------|
| `create_before_destroy = true` | Resources that can't have downtime during replacement (RDS, ELB) |
| `prevent_destroy = true` | Critical production databases, S3 buckets with data |
| `ignore_changes` | Auto-scaling managed attributes, tags managed by AWS |

```bash
# Even with prevent_destroy, this will error:
terraform destroy
# Error: Instance cannot be destroyed
# Remove prevent_destroy from config first to destroy
```
{{< /qa >}}

{{< qa num="8" q="What is the difference between terraform refresh, plan, and apply in terms of state behavior?" level="intermediate" >}}
**Ans:**

| Command | Reads real infra? | Updates state? | Makes changes? |
|---------|------------------|---------------|----------------|
| `terraform refresh` | Yes | Yes | No |
| `terraform plan` | Yes (by default) | No | No (preview only) |
| `terraform apply` | Yes | Yes | Yes |

```bash
# terraform refresh (deprecated, now use plan -refresh-only)
# Queries actual AWS resources and updates state file to match reality
# Does NOT modify real infrastructure
terraform refresh

# terraform plan
# 1. Reads .tf config
# 2. Reads state file
# 3. Queries real infra (refresh)
# 4. Shows diff: what will be created/changed/destroyed
terraform plan
terraform plan -refresh-only   # Only show drift, don't plan config changes

# terraform apply
# 1. Runs plan
# 2. User approves
# 3. Makes changes to real infrastructure
# 4. Updates state file
terraform apply
terraform apply -auto-approve  # Skip interactive approval
```

**Scenario where plan shows no change but apply modifies:**
- Provider has a bug where it always updates a resource on apply
- Race condition where another process changed the resource between plan and apply
- Non-deterministic computed values (random IDs, timestamps)
{{< /qa >}}

{{< qa num="9" q="What is the difference between terraform import and terraform taint?" level="intermediate" >}}
**Ans:**

**`terraform import`** — Brings an existing real resource under Terraform management by adding it to the state file.

```bash
# Syntax: terraform import <resource_type>.<name> <resource_id>
terraform import aws_instance.web i-1234567890abcdef0

# After import, add the resource block to your .tf file to match
# (terraform import does NOT generate .tf code automatically)

# Terraform 1.5+ supports import blocks in config:
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}
```

**`terraform taint`** — Marks a resource for forced recreation on next apply. **Deprecated in Terraform 1.x**, replaced by `-replace`.

```bash
# Old way (deprecated):
terraform taint aws_instance.web

# New way:
terraform apply -replace="aws_instance.web"

# Use case: force recreate an instance that's behaving strangely
# without changing any configuration
```

| Command | When to use |
|---------|------------|
| `import` | Resource exists in cloud but not in Terraform state |
| `-replace` | Resource exists in both but needs forced recreation |
{{< /qa >}}

{{< qa num="10" q="How do you securely manage secrets in Terraform?" level="intermediate" >}}
**Ans:**

**The problem:** Secrets passed to Terraform often end up in `.tfstate` (which is stored in S3).

```hcl
# BAD - hardcoded secret visible in code
resource "aws_db_instance" "db" {
  password = "mysecretpassword"   # ← exposed in state file and git!
}
```

**Solutions:**

```bash
# Method 1: Environment variables (not stored in state)
export TF_VAR_db_password="mysecret"
# Reference in .tf:
# variable "db_password" { sensitive = true }

# Method 2: AWS Secrets Manager data source
data "aws_secretsmanager_secret_version" "db_pass" {
  secret_id = "prod/db-password"
}
resource "aws_db_instance" "db" {
  password = data.aws_secretsmanager_secret_version.db_pass.secret_string
}

# Method 3: AWS SSM Parameter Store
data "aws_ssm_parameter" "db_pass" {
  name            = "/prod/db-password"
  with_decryption = true
}

# Method 4: Vault Provider
data "vault_generic_secret" "db" {
  path = "secret/prod/db"
}

# Method 5: Mark variable as sensitive (hides from plan output)
variable "db_password" {
  type      = string
  sensitive = true
}

# Encrypt state at rest (S3 backend)
backend "s3" {
  encrypt = true
  kms_key_id = "arn:aws:kms:..."
}
```
{{< /qa >}}

{{< qa num="11" q="What are the risks of manually editing the .tfstate file? How do you recover from corruption?" level="advanced" >}}
**Ans:**

**Risks of manual editing:**
- JSON syntax errors → Terraform completely broken
- Wrong resource IDs → Terraform destroys and recreates resources
- Missing attributes → plan/apply errors
- Inconsistent state → drift between state and reality

**Prevention:**
```bash
# Never edit state directly. Use official commands:
terraform state mv       # Rename/move resources in state
terraform state rm       # Remove resource from state (without destroying)
terraform state pull     # Download state from remote
terraform state push     # Upload state to remote
terraform import         # Add existing resource to state
```

**Recovery from corruption:**

```bash
# Step 1: Pull current state and inspect
terraform state pull > backup-state.json
cat backup-state.json | python3 -m json.tool | head -50

# Step 2: If S3 backend with versioning enabled:
aws s3api list-object-versions \
  --bucket my-tf-state-bucket \
  --prefix prod/terraform.tfstate

# Restore previous version:
aws s3api copy-object \
  --bucket my-tf-state-bucket \
  --copy-source my-tf-state-bucket/prod/terraform.tfstate?versionId=PREV_VERSION \
  --key prod/terraform.tfstate

# Step 3: If state is completely lost - rebuild
# Run terraform import for each resource
terraform import aws_instance.web i-1234567890abcdef

# Step 4: Validate restored state
terraform plan
# Should show no changes if state matches reality
```

**Best practices to prevent:**
- Enable S3 bucket versioning
- Never edit `.tfstate` manually — use `terraform state` commands
- Lock state with DynamoDB
{{< /qa >}}

{{< qa num="12" q="How would you migrate Terraform state from local backend to remote backend without downtime?" level="advanced" >}}
**Ans:**

```bash
# Step 1: Ensure state is not currently in use (no apply running)

# Step 2: Backup local state
cp terraform.tfstate terraform.tfstate.backup

# Step 3: Create S3 bucket and DynamoDB table for remote backend
aws s3api create-bucket --bucket my-tf-state-prod --region us-east-1
aws s3api put-bucket-versioning \
  --bucket my-tf-state-prod \
  --versioning-configuration Status=Enabled

aws dynamodb create-table \
  --table-name tf-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Step 4: Add backend config to main.tf
terraform {
  backend "s3" {
    bucket         = "my-tf-state-prod"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "tf-state-lock"
  }
}

# Step 5: Run terraform init — it will ask to migrate state
terraform init
# "Do you want to copy existing state to the new backend? yes"

# Step 6: Verify state was migrated
terraform state list

# Step 7: Confirm local state is no longer active
# terraform.tfstate is now empty/minimal

# Step 8: Test with a plan
terraform plan   # Should show no changes
```
{{< /qa >}}

{{< qa num="13" q="How do you refactor Terraform code without destroying existing infrastructure?" level="advanced" >}}
**Ans:**

**Scenario: Renaming a resource, moving to a module, or restructuring without destroy.**

```bash
# Method 1: terraform state mv (rename/restructure in state)
# Before: resource "aws_instance" "old_name"
# After:  resource "aws_instance" "new_name"

terraform state mv aws_instance.old_name aws_instance.new_name

# Method 2: moved blocks (Terraform 1.1+, declarative, version-controlled)
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}
# Run: terraform plan → shows "moved" not "create/destroy"
# Apply → state updated, no infrastructure change

# Method 3: Moving resource into a module
moved {
  from = aws_instance.web
  to   = module.webserver.aws_instance.web
}

# Method 4: Use -replace only for deliberate recreation
# terraform apply -replace="aws_instance.web"

# Always verify with plan before apply:
terraform plan   # Must show 0 changes to destroy for safe refactor
```

**Golden rule:** After refactoring, `terraform plan` should show `Plan: 0 to add, 0 to change, 0 to destroy`.
{{< /qa >}}

{{< qa num="14" q="How would you handle a situation where Terraform partially created resources and failed midway?" level="advanced" >}}
**Ans:**

```bash
# Terraform partial creates - what happens:
# - Resources created BEFORE failure → in state file
# - Resources that failed → may be partially in state or not

# Step 1: Check current state
terraform state list

# Step 2: Check what was actually created in AWS
aws ec2 describe-instances --filters Name=tag:Environment,Values=prod

# Step 3: Run terraform plan to see current diff
terraform plan
# Terraform will show: resources to create (that failed), changes needed

# Step 4: Simply re-run apply
terraform apply
# Terraform is idempotent — it only creates missing resources
# Already-created resources are left alone

# Step 5: If a resource is in a broken half-created state:
# Option A: Remove from state, destroy manually, re-apply
terraform state rm aws_resource.broken
aws <service> delete-<resource> --id <id>
terraform apply

# Option B: Import the broken resource and fix
terraform import aws_resource.broken <resource-id>
# Fix the config and apply

# Step 6: If the error was in a provider/dependency:
# Fix the code and re-apply — Terraform figures out what's missing
```

**Terraform is idempotent by design** — running `apply` multiple times is safe. It will only create/update resources that don't match desired state.
{{< /qa >}}

{{< qa num="15" q="How do you design reusable, version-controlled Terraform modules for enterprise use?" level="advanced" >}}
**Ans:**

```
repo: terraform-modules/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── ec2/
│   │   ├── main.tf
│   │   └── ...
│   └── rds/
│       └── ...
```

```hcl
# modules/ec2/variables.tf
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Must be a valid instance type"
  }
}

# modules/ec2/main.tf
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  
  tags = merge(var.tags, {
    ManagedBy = "Terraform"
  })
}

# modules/ec2/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}
```

```hcl
# environments/prod/main.tf - Consumer
module "webserver" {
  source  = "git::https://github.com/myorg/terraform-modules.git//modules/ec2?ref=v1.2.0"
  
  instance_type = "t3.medium"
  ami           = "ami-xxx"
  subnet_id     = module.vpc.public_subnet_id
  tags          = { Environment = "prod" }
}
```

**Enterprise best practices:**
- Pin module versions with `?ref=v1.2.0` (never `main` branch)
- Use Terraform Registry for public/private module hosting
- Validate with `terraform validate` + `tflint` in CI
- Test modules with `terratest`
- Document with `terraform-docs`
{{< /qa >}}

{{< qa num="16" q="How do terraform workspaces work and why are they dangerous in large organizations?" level="advanced" >}}
**Ans:**

**Terraform workspaces** allow multiple state files within the same backend configuration, accessed via a workspace name.

```bash
# Create and switch workspaces
terraform workspace new dev
terraform workspace new prod
terraform workspace list
terraform workspace select prod

# In config, use workspace name
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
}

# State files:
# s3://bucket/key.tfstate          (default workspace)
# s3://bucket/env:/dev/key.tfstate
# s3://bucket/env:/prod/key.tfstate
```

**Why workspaces are dangerous in large orgs:**

1. **Single codebase** — all environments share same `.tf` files. A mistake in code affects ALL workspaces.
2. **No isolation** — developer on `dev` workspace can accidentally `select prod` and destroy prod.
3. **Different backends not supported** — can't use different S3 buckets/regions per workspace easily.
4. **Poor visibility** — hard to see which workspace you're in → human error.

**Better alternative: Separate directories per environment:**
```
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── backend.tf   # different S3 key
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...           # completely separate state
```
{{< /qa >}}

{{< qa num="17" q="How do you manage secrets in Terraform while working in teams?" level="intermediate" >}}
**Ans:**

```bash
# Problem: State file stored in S3 contains secrets in plaintext unless encrypted

# Solution 1: Encrypt state at rest
terraform {
  backend "s3" {
    bucket     = "tf-state"
    key        = "prod/terraform.tfstate"
    encrypt    = true                                   # AES-256 or KMS
    kms_key_id = "arn:aws:kms:us-east-1:123:key/xxx"  # Optional KMS key
  }
}

# Solution 2: Use sensitive = true for variables
variable "db_password" {
  type      = string
  sensitive = true     # Masked in plan/apply output
}

# Solution 3: Never store secrets in .tf files or tfvars committed to git
# Use .gitignore:
echo "*.tfvars" >> .gitignore
echo "*.tfstate" >> .gitignore

# Solution 4: Pass secrets via environment variables
export TF_VAR_db_password="$(aws secretsmanager get-secret-value \
  --secret-id prod/db --query SecretString --output text)"

# Solution 5: HashiCorp Vault integration
provider "vault" {}
data "vault_generic_secret" "db" {
  path = "secret/prod/database"
}
```

**Team workflow:**
- Secrets in Vault or AWS Secrets Manager
- CI/CD pipeline (Jenkins/GitHub Actions) fetches secrets at runtime
- State in encrypted S3 with access limited by IAM
- No secrets ever committed to Git
{{< /qa >}}

{{< qa num="18" q="Explain depends_on vs implicit dependency — when does Terraform get it wrong?" level="advanced" >}}
**Ans:**

**Implicit dependency** — Terraform automatically detects when one resource references another's attribute. It builds a dependency graph.

```hcl
resource "aws_vpc" "main" {}

resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id   # Implicit: subnet depends on VPC
  # Terraform knows VPC must be created first
}
```

**Explicit `depends_on`** — Forces dependency when there's no attribute reference (hidden dependencies).

```hcl
resource "aws_s3_bucket" "data" {}

resource "aws_s3_bucket_policy" "policy" {
  bucket = aws_s3_bucket.data.id
  policy = data.aws_iam_policy_document.allow.json
}

resource "aws_lambda_function" "processor" {
  ...
  depends_on = [aws_s3_bucket_policy.policy]
  # Lambda must only be created after policy is applied
  # No attribute reference → need explicit depends_on
}
```

**When Terraform gets it wrong:**
```hcl
# Scenario: IAM role created, but EC2 still fails because
# IAM eventual consistency hasn't propagated yet
resource "aws_iam_role" "role" {}
resource "aws_iam_instance_profile" "profile" {
  role = aws_iam_role.role.name
}

# AWS IAM is eventually consistent — role may not be usable immediately
# Use explicit depends_on + time_sleep
resource "time_sleep" "wait_for_iam" {
  depends_on      = [aws_iam_role.role]
  create_duration = "15s"
}

resource "aws_instance" "web" {
  depends_on           = [time_sleep.wait_for_iam]
  iam_instance_profile = aws_iam_instance_profile.profile.name
}
```
{{< /qa >}}

</div>
