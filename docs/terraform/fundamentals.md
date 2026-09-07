# Terraform Fundamentals

## Overview

Terraform is an **Infrastructure as Code (IaC)** tool used to define and manage infrastructure through configuration files.

The Employee Management API project uses Terraform to define and manage its AWS infrastructure.

This page provides a technical reference for the Terraform fundamentals learned during the early stages of the project.

---

## 1. Infrastructure as Code

Infrastructure as Code means managing infrastructure through machine-readable configuration instead of relying entirely on manual configuration.

Instead of manually creating AWS resources through the AWS Console, Terraform configuration describes the desired infrastructure.

```text
Terraform Configuration
          ↓
       Terraform
          ↓
     AWS Provider
          ↓
    AWS Infrastructure
```

### Benefits

* Version-controlled infrastructure
* Repeatable deployments
* Reviewable changes
* Reduced manual configuration
* Reusable infrastructure
* Easier environment management

---

## 2. Declarative Configuration

Terraform uses a declarative approach.

The configuration describes the desired state rather than providing a step-by-step procedure for creating the infrastructure.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}
```

Terraform determines the actions required to make the actual infrastructure match the configuration.

---

## 3. Providers

Providers allow Terraform to interact with external platforms and services.

The Employee Management API uses the AWS provider.

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.40"
    }
  }
}
```

Provider configuration:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

The first configuration specifies the provider dependency and version constraint.

The second configures the provider itself.

---

## 4. Resources

Resources represent infrastructure objects managed by Terraform.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

A resource consists of:

```text
Resource type
      +
Local resource name
      +
Configuration arguments
```

The resource can later be referenced using:

```hcl
aws_dynamodb_table.employee
```

---

## 5. Variables

Variables provide configurable input values.

Declaration:

```hcl
variable "table_name" {
  type        = string
  description = "Name of the DynamoDB table"
}
```

Usage:

```hcl
name = var.table_name
```

Value:

```hcl
table_name = "employee-dev-table"
```

Variables help separate infrastructure logic from environment-specific configuration.

---

## 6. Outputs

Outputs expose useful information from Terraform.

Example:

```hcl
output "dynamodb_table_name" {
  value = aws_dynamodb_table.employee.name
}
```

View outputs:

```bash
terraform output
```

Variables and outputs provide opposite directions of information flow:

```text
Variables
    ↓
Input to Terraform

Terraform
    ↓
Outputs
    ↓
Useful infrastructure information
```

---

## 7. Terraform Project Structure

A simple Terraform project can be organized as:

```text
terraform/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

These filenames are conventions. Terraform loads `.tf` files in the same directory as one configuration.

As infrastructure grows, the project can evolve into reusable modules and environment-specific directories.

---

## 8. Terraform Workflow

The basic Terraform workflow is:

```text
Write Configuration
        ↓
terraform init
        ↓
terraform plan
        ↓
Review Plan
        ↓
terraform apply
        ↓
Infrastructure Updated
```

### `terraform init`

Initializes the Terraform working directory.

It installs required providers and prepares the working directory.

### `terraform plan`

Generates a proposed execution plan.

It allows changes to be reviewed before they are applied.

### `terraform apply`

Applies the configuration and makes the required infrastructure changes.

---

## 9. Terraform in the Employee Management API

Terraform manages the infrastructure required by the Employee Management API.

The architecture eventually included:

```text
              API Gateway
                   │
                   ▼
                Lambda
                   │
                   ▼
               DynamoDB
```

Supporting infrastructure included:

```text
IAM
KMS
CloudWatch
```

Terraform became the source of truth for the infrastructure configuration.

---

## 10. Terraform Architecture Evolution

The project's Terraform architecture evolved progressively.

```text
Terraform Fundamentals
        ↓
Project Structure
        ↓
Providers & Resources
        ↓
Variables
        ↓
Outputs
        ↓
Dependencies
        ↓
Reusable Modules
        ↓
Environment Separation
        ↓
Remote State
        ↓
CI/CD
        ↓
Advanced Infrastructure Engineering
```

This evolution reflects how the project became more structured as new infrastructure requirements were introduced.

---

## 11. Core Terraform Concepts

| Concept     | Purpose                                                      |
| ----------- | ------------------------------------------------------------ |
| Provider    | Connects Terraform to external platforms                     |
| Resource    | Represents infrastructure managed by Terraform               |
| Variable    | Provides configurable input                                  |
| Output      | Exposes useful infrastructure information                    |
| Module      | Packages reusable Terraform configuration                    |
| State       | Tracks Terraform-managed infrastructure                      |
| Backend     | Defines where Terraform state is stored                      |
| Data Source | Reads existing information without managing it as a resource |

Modules, state, backends, and data sources were covered more deeply in later stages of the learning journey.

---

## 12. Common Terraform Commands

### Initialize

```bash
terraform init
```

### Validate

```bash
terraform validate
```

### Format

```bash
terraform fmt
```

### Generate a plan

```bash
terraform plan
```

### Apply changes

```bash
terraform apply
```

### View outputs

```bash
terraform output
```

### Show state

```bash
terraform show
```

These commands form the basic Terraform workflow and were used as the project evolved.

---

## 13. Key Principles

### Declarative

Describe the desired infrastructure state.

### Version Controlled

Keep infrastructure configuration in Git.

### Reviewable

Use plans and pull requests to review infrastructure changes.

### Reusable

Use variables and modules to avoid unnecessary duplication.

### Reproducible

Infrastructure should be defined in a way that allows it to be recreated consistently.

### Progressive Architecture

Start simple and introduce additional architecture only when the project requires it.

---

## 14. Learning Progression

The early Terraform learning path can be summarized as:

```text
Sprint 1
Terraform Introduction
        ↓
Sprint 2
Project Structure
        ↓
Sprint 3
Providers & Resources
        ↓
Sprint 4
Variables
        ↓
Sprint 5
Outputs
```

These fundamentals prepared the project for the next stage:

```text
Sprint 6
Dependencies
        ↓
Sprint 7
Modules
        ↓
Sprint 8
Reusable Infrastructure
```

---

## 15. Summary

The core Terraform concepts established during the first five sprints were:

* Infrastructure as Code
* Declarative infrastructure
* Providers
* Resources
* Variables
* Outputs
* Terraform project organization
* Basic Terraform workflow

These concepts formed the foundation for the Employee Management API's later Terraform architecture, including reusable modules, environment separation, remote state, CI/CD, and advanced infrastructure engineering.
