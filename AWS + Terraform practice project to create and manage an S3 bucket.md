Here's a hands-on AWS + Terraform practice project to create and manage an S3 bucket.

# Project: Launch an S3 Bucket Using Terraform

## Prerequisites

* AWS Account
* IAM User with S3 permissions
* Terraform installed
* AWS CLI installed and configured

Verify:

```bash
terraform -version
aws --version
aws sts get-caller-identity
```

---

## Project Structure

```text
terraform-s3-project/
│
├── provider.tf
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

---

## Step 1: provider.tf

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

---

## Step 2: variables.tf

```hcl
variable "aws_region" {
  default = "us-east-1"
}

variable "bucket_name" {
  description = "Unique S3 Bucket Name"
}
```

---

## Step 3: terraform.tfvars

Replace with a globally unique bucket name.

```hcl
bucket_name = "farhan-devops-terraform-s3-123456"
```

---

## Step 4: main.tf

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = var.bucket_name

  tags = {
    Name        = "Terraform-S3-Bucket"
    Environment = "Dev"
  }
}
```

---

## Step 5: outputs.tf

```hcl
output "bucket_name" {
  value = aws_s3_bucket.my_bucket.bucket
}

output "bucket_arn" {
  value = aws_s3_bucket.my_bucket.arn
}
```

---

# Deploy Infrastructure

Initialize Terraform:

```bash
terraform init
```

Validate configuration:

```bash
terraform validate
```

Preview changes:

```bash
terraform plan
```

Create bucket:

```bash
terraform apply
```

Type:

```text
yes
```

---

# Verify Bucket

AWS CLI:

```bash
aws s3 ls
```

or check in the AWS Console:

Amazon Web Services S3 → Buckets

---

# Upload a File

Create a test file:

```bash
echo "Terraform Practice" > test.txt
```

Upload:

```bash
aws s3 cp test.txt s3://farhan-devops-terraform-s3-123456/
```

Verify:

```bash
aws s3 ls s3://farhan-devops-terraform-s3-123456/
```

---

# Bonus Tasks (Interview-Oriented)

### 1. Enable Versioning

```hcl
resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.my_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

---

### 2. Enable Server-Side Encryption

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "encryption" {
  bucket = aws_s3_bucket.my_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

---

### 3. Block Public Access

```hcl
resource "aws_s3_bucket_public_access_block" "block" {
  bucket = aws_s3_bucket.my_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

---

### 4. Create a Folder

```hcl
resource "aws_s3_object" "folder" {
  bucket = aws_s3_bucket.my_bucket.id
  key    = "documents/"
}
```

---

### 5. Upload a File Using Terraform

```hcl
resource "aws_s3_object" "sample_file" {
  bucket = aws_s3_bucket.my_bucket.id
  key    = "documents/readme.txt"
  source = "readme.txt"
}
```

---

# Cleanup

Delete everything:

```bash
terraform destroy
```

This is a good beginner-to-intermediate Terraform project because it covers:

* Provider configuration
* Variables
* Outputs
* S3 bucket creation
* Versioning
* Encryption
* Public access blocking
* File upload
* Infrastructure cleanup

These are common topics asked in Terraform and AWS DevOps interviews.
