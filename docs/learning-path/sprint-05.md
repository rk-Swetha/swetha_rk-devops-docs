# Sprint 5 — Terraform Outputs

## Overview

Sprint 5 focused on Terraform output values.

Outputs allow Terraform to expose useful information from managed resources so that it can be viewed after Terraform operations or consumed by other parts of the Terraform configuration.

Outputs became particularly important later when the project introduced reusable modules.

---

## 1. What are Terraform Outputs?

An output value exposes information from Terraform configuration.

A basic output looks like:

```hcl
output "dynamodb_table_name" {
  value = aws_dynamodb_table.employee.name
}
```

After Terraform applies the configuration, the output can be displayed using:

```bash
terraform output
```

---

## 2. Output Syntax

The basic structure is:

```hcl
output "output_name" {
  value = expression
}
```

For example:

```hcl
output "lambda_arn" {
  value = aws_lambda_function.employee.arn
}
```

Here:

```text
lambda_arn
    ↓
Output name

aws_lambda_function.employee.arn
    ↓
Value being exposed
```

---

## 3. Accessing Resource Attributes

Outputs commonly expose attributes of Terraform resources.

For example:

```hcl
output "dynamodb_table_name" {
  value = aws_dynamodb_table.employee.name
}
```

The output retrieves the `name` attribute from the DynamoDB resource.

Another example:

```hcl
output "lambda_arn" {
  value = aws_lambda_function.employee.arn
}
```

This exposes the Lambda function ARN.

---

## 4. Why Outputs Are Useful

Outputs can be useful for:

* Displaying important infrastructure information.
* Connecting modules.
* Passing information to other Terraform configurations.
* Obtaining resource identifiers.
* Supporting automation and deployment workflows.

For example, an API Gateway module may expose an API URL:

```hcl
output "api_url" {
  value = ...
}
```

The root module can then consume that output.

---

## 5. Outputs and Modules

Outputs became especially important when the project introduced child modules.

A module can expose values through outputs.

Conceptually:

```text
Child Module
     │
     │ output
     ▼
Root Module
     │
     │ input
     ▼
Another Module
```

For example:

```text
DynamoDB Module
      │
      │ table ARN
      ▼
Root Module
      │
      │ table ARN
      ▼
IAM / Lambda Module
```

This allows modules to communicate without exposing their internal implementation details.

---

## 6. Project Outputs

The Employee Management API project used outputs including:

```text
api_url
dynamodb_table_name
iam_role
lambda_arn
```

These outputs provide useful information about the deployed infrastructure.

---

## 7. Example Project Outputs

### API URL

```hcl
output "api_url" {
  value = ...
}
```

This exposes the API Gateway endpoint.

### DynamoDB Table Name

```hcl
output "dynamodb_table_name" {
  value = ...
}
```

This exposes the DynamoDB table name.

### IAM Role

```hcl
output "iam_role" {
  value = ...
}
```

This exposes information about the IAM role.

### Lambda ARN

```hcl
output "lambda_arn" {
  value = ...
}
```

This exposes the Lambda function ARN.

The exact expression depends on which module or resource owns the value.

---

## 8. Viewing Outputs

After Terraform has applied the configuration, outputs can be viewed with:

```bash
terraform output
```

A specific output can be queried using:

```bash
terraform output api_url
```

This is useful when infrastructure creates values that are not convenient to retrieve manually.

---

## 9. Outputs vs Variables

Variables and outputs serve opposite directions.

### Variables

Values enter Terraform:

```text
Configuration
      ↓
Variable
      ↓
Terraform
```

### Outputs

Values leave Terraform:

```text
Terraform
    ↓
Output
    ↓
User / Module / Automation
```

A simple way to remember:

> **Variables are inputs. Outputs are exposed results.**

---

## 10. Application to Employee Management API

Outputs were used to expose important information from the Employee Management API infrastructure.

The project eventually exposed:

```text
API URL
DynamoDB table name
IAM role
Lambda ARN
```

As modules were introduced, outputs became part of the interface between the root configuration and child modules.

This allowed the Terraform architecture to become more modular and reusable.

---

## 11. Learned vs Implemented

### 📚 Learned

* Terraform outputs
* Output syntax
* Resource attributes
* `terraform output`
* Referencing outputs
* Outputs in modules
* Module communication

### 🛠️ Implemented

The Employee Management API Terraform configuration exposed useful infrastructure information through outputs.

Actual project outputs included:

* `api_url`
* `dynamodb_table_name`
* `iam_role`
* `lambda_arn`

Outputs later became part of the module architecture.

### 🔮 Future Improvements

Outputs can be further improved by:

* Clearly documenting output descriptions.
* Marking appropriate sensitive outputs as sensitive.
* Keeping module interfaces minimal.
* Avoiding unnecessary exposure of internal resource details.

---

## 12. Key Takeaways

> Terraform outputs expose useful information from infrastructure.

> Outputs can reference resource attributes.

> `terraform output` can be used to view output values.

> Variables provide inputs to Terraform, while outputs expose results from Terraform.

> Outputs become especially important when building reusable Terraform modules.

Sprint 5 completed the initial Terraform fundamentals and prepared the project for the next stage: **Terraform dependencies and reusable modules**.
