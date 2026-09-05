# Sprint 1 — Terraform Introduction

## Overview

Sprint 1 introduced the fundamentals of **Terraform** and the concept of **Infrastructure as Code (IaC)**.

The main objective was to understand how infrastructure can be defined, managed, and reproduced using configuration files instead of manually creating resources through the AWS Management Console.

Terraform became the foundation for building the infrastructure of the **Employee Management API**.

---

## 1. What is Infrastructure as Code?

**Infrastructure as Code (IaC)** is the practice of defining and managing infrastructure using machine-readable configuration files.

Instead of manually creating AWS resources through the AWS Console, infrastructure can be described using code.

For example, instead of manually creating a DynamoDB table, Terraform can define it:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}
```

Terraform reads this configuration and creates the required infrastructure.

---

## 2. Why Terraform?

Before learning Terraform, infrastructure could be created manually through the AWS Console.

Manual infrastructure management can become difficult when:

* There are many AWS resources.
* Multiple environments are required.
* Infrastructure needs to be recreated.
* Multiple engineers work on the same infrastructure.
* Changes need to be reviewed and tracked.
* Infrastructure configuration needs to be version controlled.

Terraform addresses these problems by allowing infrastructure to be represented as code.

### Key advantages learned

* Infrastructure can be version controlled using Git.
* Infrastructure changes can be reviewed through pull requests.
* The same configuration can be reused across environments.
* Terraform can identify changes before applying them.
* Infrastructure becomes more reproducible.
* Infrastructure configuration becomes easier to maintain.

---

## 3. Terraform vs Manual AWS Console

| Manual AWS Console                         | Terraform                                                  |
| ------------------------------------------ | ---------------------------------------------------------- |
| Resources created manually                 | Resources defined as code                                  |
| Difficult to reproduce exactly             | Configuration can be reproduced                            |
| Changes are harder to track                | Changes can be tracked in Git                              |
| Manual configuration                       | Declarative configuration                                  |
| Difficult to review infrastructure changes | Changes can be reviewed using `terraform plan` and Git PRs |
| Repetition across environments             | Same modules/configuration can be reused                   |

Terraform does not completely replace the AWS Console. The console remains useful for inspecting resources, troubleshooting, and understanding the AWS environment.

However, Terraform becomes the **source of truth for infrastructure configuration** in the project.

---

## 4. Terraform's Declarative Approach

Terraform uses a **declarative model**.

Instead of telling Terraform every individual step required to create infrastructure, we describe the **desired end state**.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}
```

The configuration describes what should exist.

Terraform determines the actions required to make the actual infrastructure match that desired configuration.

This is different from an imperative approach where we would explicitly describe each step to execute.

---

## 5. Terraform Workflow

The basic Terraform workflow learned during this sprint was:

```text
Write Terraform Configuration
          ↓
terraform init
          ↓
terraform plan
          ↓
Review Changes
          ↓
terraform apply
          ↓
Infrastructure Created/Updated
```

### `terraform init`

Initializes the Terraform working directory.

It downloads required providers and prepares Terraform to work with the configuration.

### `terraform plan`

Creates an execution plan showing what Terraform intends to change.

This allows infrastructure changes to be reviewed before applying them.

### `terraform apply`

Applies the planned changes and creates or updates the infrastructure.

---

## 6. Providers

Terraform itself does not directly know how to communicate with AWS, Azure, Google Cloud, or other platforms.

It uses **providers** to interact with external platforms and services.

For this project, the **AWS provider** was used.

Example:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

The AWS provider allows Terraform to manage AWS resources such as:

* Lambda
* API Gateway
* DynamoDB
* IAM
* KMS
* CloudWatch

---

## 7. Resources

A Terraform **resource** represents an infrastructure object that Terraform manages.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}
```

In this example:

* `aws_dynamodb_table` → resource type
* `employee` → Terraform resource name
* `name` → AWS DynamoDB table name
* `billing_mode` → table billing configuration
* `hash_key` → partition key

Terraform uses resource definitions to understand what infrastructure should exist.

---

## 8. Application to the Employee Management API

Terraform was introduced as the Infrastructure as Code solution for the Employee Management API.

The initial infrastructure direction was:

```text
Employee Management API
        │
        ▼
    Terraform
        │
        ├── API Gateway
        ├── Lambda
        ├── DynamoDB
        ├── IAM
        ├── KMS
        └── CloudWatch
```

Instead of manually creating these AWS resources through the AWS Console, the infrastructure was progressively defined using Terraform.

This established the foundation for later topics such as:

* Variables
* Outputs
* Dependencies
* Modules
* Environment separation
* Remote state
* State locking
* CI/CD
* Infrastructure security
* Production readiness

---

## 9. What Was Learned vs Implemented

### 📚 Learned

* Infrastructure as Code
* Terraform fundamentals
* Declarative infrastructure
* Terraform workflow
* Providers
* Resources
* Basic Terraform syntax
* Terraform vs manual infrastructure management

### 🛠️ Implemented

Terraform was adopted as the infrastructure management tool for the Employee Management API.

The project's AWS infrastructure was progressively managed through Terraform rather than relying on manual AWS Console configuration.

### 🔮 Future Improvements

At this stage, the project was still developing its Terraform architecture.

Later sprints introduced:

* Reusable modules
* Environment separation
* Remote state
* State locking
* State recovery
* CI/CD automation
* Security validation
* Multi-region strategies
* Disaster recovery
* Production-readiness practices

---

## 10. Key Takeaways

The most important concepts from Sprint 1 were:

> **Terraform allows infrastructure to be defined as code.**

> **Terraform uses a declarative approach: describe the desired state and Terraform determines how to reach it.**

> **Providers allow Terraform to interact with platforms such as AWS.**

> **Resources represent infrastructure objects managed by Terraform.**

> **`terraform plan` allows changes to be reviewed before they are applied.**

Sprint 1 established the foundation for all subsequent Terraform learning and the infrastructure architecture of the Employee Management API.