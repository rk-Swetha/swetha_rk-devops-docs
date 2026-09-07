# Sprint 2 — Terraform Project Structure

## Overview

Sprint 2 focused on organizing a Terraform project into separate configuration files.

The goal was to understand how Terraform configuration can be structured logically instead of keeping the entire infrastructure definition in a single file.

This sprint established the initial project structure that was later expanded into reusable modules and environment-specific configurations.

---

## 1. Terraform Configuration Files

Terraform configuration is commonly organized into multiple `.tf` files based on responsibility.

Terraform loads all `.tf` files within the same directory as a single configuration. The separation into files is primarily for organization and maintainability.

The initial project structure was:

```text
terraform/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

---

## 2. `main.tf`

`main.tf` is commonly used for the main infrastructure configuration.

It can contain resource definitions and module calls.

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

The filename itself does not have special meaning to Terraform. It is a convention used to make the configuration easier to understand.

---

## 3. `provider.tf`

`provider.tf` was used to define the AWS provider configuration.

Example:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

The provider allows Terraform to communicate with AWS and manage AWS infrastructure.

Provider version requirements are defined separately in the Terraform configuration.

---

## 4. `variables.tf`

`variables.tf` contains input variable declarations.

Example:

```hcl
variable "table_name" {
  type        = string
  description = "Name of the DynamoDB table"
}
```

Variables prevent infrastructure configuration from being unnecessarily hardcoded.

Instead of writing:

```hcl
name = "employee-dev-table"
```

the configuration can use:

```hcl
name = var.table_name
```

---

## 5. `terraform.tfvars`

`terraform.tfvars` contains values for input variables.

Example:

```hcl
table_name = "employee-dev-table"
```

This separates environment-specific or configurable values from the infrastructure logic.

---

## 6. `outputs.tf`

`outputs.tf` defines values that Terraform should expose after creating or updating infrastructure.

Example:

```hcl
output "dynamodb_table_name" {
  value = aws_dynamodb_table.employee.name
}
```

Outputs became particularly useful later when modules were introduced because one module could expose information required by another module.

---

## 7. `.gitignore`

Terraform generates local files that should generally not be committed to Git.

A Terraform `.gitignore` can include:

```text
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
crash.log
```

Care should be taken with `terraform.tfvars` because variable files may contain sensitive values.

Terraform state files can also contain sensitive information and should not normally be committed to source control.

---

## 8. Why Separate Files?

Separating configuration by responsibility makes the project easier to understand.

For example:

```text
provider.tf
    ↓
How Terraform connects to AWS

variables.tf
    ↓
What values can be configured

main.tf
    ↓
What infrastructure should exist

outputs.tf
    ↓
What information Terraform exposes

terraform.tfvars
    ↓
What values are supplied
```

This is primarily an organizational convention. Terraform treats all `.tf` files in the same directory as one configuration.

---

## 9. Initial Project Structure

At the beginning of the project, the structure was intentionally simple:

```text
terraform/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

This structure was suitable while learning Terraform fundamentals.

As the infrastructure became more complex, keeping everything in one root configuration would have become harder to maintain.

---

## 10. Evolution of the Project Structure

The project was later refactored into reusable modules and environment-specific configurations.

The evolved structure became:

```text
employee-management-api/
│
├── terraform/
│   ├── modules/
│   │   ├── api_gateway/
│   │   ├── dynamodb/
│   │   ├── iam/
│   │   ├── kms/
│   │   └── lambda/
│   │
│   └── environments/
│       └── dev/
│
├── terraform-bootstrap/
│   ├── main.tf
│   ├── variables.tf
│   ├── locals.tf
│   └── terraform.tfvars
│
├── src/
│   └── app.py
│
└── .github/
    └── workflows/
        └── terraform-ci.yml
```

This was not the structure used when Terraform was first introduced.

It evolved as new requirements were introduced:

```text
Simple Terraform
       ↓
Variables
       ↓
Outputs
       ↓
Modules
       ↓
Environment separation
       ↓
Remote state
       ↓
CI/CD
```

---

## 11. Application to Employee Management API

The project uses Terraform to manage the infrastructure required by the Employee Management API.

The infrastructure eventually included:

* API Gateway
* Lambda
* DynamoDB
* IAM
* KMS
* CloudWatch

The project structure allowed these resources to be organized and later separated into reusable modules.

---

## 12. Learned vs Implemented

### 📚 Learned

* Terraform configuration files
* Terraform project organization
* Input variables
* Outputs
* Provider configuration
* Separation of configuration responsibilities
* Terraform project structure

### 🛠️ Implemented

The Employee Management API started with a simple Terraform structure using separate configuration files.

The structure was later expanded into modules and environment-specific configurations as the project evolved.

### 🔮 Future Improvements

Further improvements can include:

* Additional environment directories
* More reusable modules
* Automated documentation validation
* Additional CI/CD checks

---

## 13. Key Takeaways

> Terraform loads all `.tf` files in a directory as a single configuration.

> File names such as `main.tf`, `variables.tf`, and `outputs.tf` are organizational conventions.

> Separating configuration by responsibility improves readability and maintainability.

> Terraform project structure should evolve as infrastructure complexity increases.

> The Employee Management API started with a simple structure and later evolved into reusable modules and environment-specific configurations.
