# Sprint 8 — Terraform Module Inputs and Outputs

## Overview

Terraform modules become truly reusable when they can accept values from the calling configuration and return useful values back to it.

This is achieved using:

* **Input variables**
* **Module outputs**

A module should generally avoid hardcoding environment-specific values.

Instead, the calling configuration provides those values as inputs.

The module then creates the required infrastructure and exposes important resource information through outputs.

The overall flow is:

```text
Environment / Root Module
          │
          │ Input Variables
          ▼
     Child Module
          │
          │ Resource Configuration
          ▼
      AWS Resource
          │
          │ Output Values
          ▼
Environment / Root Module
```

This pattern is fundamental to building reusable Terraform infrastructure.

---

## 1. What Are Module Inputs?

Module inputs are values passed from a parent module to a child module.

They are defined using Terraform `variable` blocks inside the child module.

For example:

```hcl
variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
}
```

The module can then use the variable:

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

The value is supplied by the parent module.

---

## 2. Passing Inputs to a Module

Suppose the DynamoDB module defines:

```hcl
variable "table_name" {
  type = string
}
```

The root module can pass a value:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
}
```

The flow is:

```text
"employee-dev-table"
        │
        ▼
module.dynamodb
        │
        ▼
var.table_name
        │
        ▼
DynamoDB table name
```

The child module does not need to know whether the value belongs to development, testing, or production.

It simply receives the value.

---

## 3. Why Inputs Are Important

Hardcoding values inside modules reduces reusability.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

This module is tightly coupled to the development environment.

A better approach is:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = var.table_name
}
```

Now the same module can be used for:

```text
Development
    ↓
employee-dev-table

Testing
    ↓
employee-test-table

Production
    ↓
employee-prod-table
```

The module implementation remains unchanged.

Only the input changes.

---

## 4. Types of Terraform Variables

Terraform supports several commonly used variable types.

### String

```hcl
variable "table_name" {
  type = string
}
```

Example:

```hcl
table_name = "employee-dev-table"
```

### Number

```hcl
variable "memory_size" {
  type = number
}
```

Example:

```hcl
memory_size = 512
```

### Boolean

```hcl
variable "enable_logging" {
  type = bool
}
```

Example:

```hcl
enable_logging = true
```

### List

```hcl
variable "allowed_actions" {
  type = list(string)
}
```

Example:

```hcl
allowed_actions = [
  "dynamodb:GetItem",
  "dynamodb:PutItem"
]
```

### Map

```hcl
variable "tags" {
  type = map(string)
}
```

Example:

```hcl
tags = {
  Environment = "dev"
  Project     = "employee-api"
}
```

Using explicit types makes Terraform configurations easier to understand and validate.

---

## 5. Variable Descriptions

Variables should have useful descriptions.

Example:

```hcl
variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
}
```

Instead of:

```hcl
variable "table_name" {
  type = string
}
```

The description helps developers understand what the variable represents.

For reusable modules, good documentation becomes increasingly important.

---

## 6. Variable Defaults

Variables can have default values.

Example:

```hcl
variable "billing_mode" {
  description = "DynamoDB billing mode"
  type        = string
  default     = "PAY_PER_REQUEST"
}
```

If the parent module does not provide a value, Terraform uses the default.

However, defaults should be used carefully.

Environment-specific or security-sensitive values should generally not be hidden behind inappropriate defaults.

---

## 7. Required Variables

A variable without a default value is generally required.

Example:

```hcl
variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
}
```

The caller must provide:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
}
```

Otherwise Terraform will report that a required value is missing.

---

## 8. Variable Validation

Terraform allows variables to have validation rules.

Example:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string

  validation {
    condition     = contains(["dev", "test", "prod"], var.environment)
    error_message = "Environment must be dev, test, or prod."
  }
}
```

This prevents invalid values from being passed to the module.

For example:

```text
dev   → valid
test  → valid
prod  → valid
stage → invalid
```

Variable validation helps catch configuration mistakes early.

---

## 9. What Are Module Outputs?

Outputs expose values from a Terraform module.

For example:

```hcl
output "table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.employee.arn
}
```

This makes the DynamoDB table ARN available to the parent module.

The parent can access it using:

```hcl
module.dynamodb.table_arn
```

---

## 10. Why Outputs Are Important

Outputs allow modules to communicate useful information to other parts of the Terraform configuration.

For example:

```text
DynamoDB Module
      │
      │ table_arn
      ▼
IAM Module
```

The IAM configuration can use:

```hcl
module.dynamodb.table_arn
```

instead of hardcoding the ARN.

This creates a clean relationship between infrastructure components.

---

## 11. Output Examples

A module can expose several useful values.

### Table Name

```hcl
output "table_name" {
  description = "Name of the DynamoDB table"
  value       = aws_dynamodb_table.employee.name
}
```

### Table ARN

```hcl
output "table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.employee.arn
}
```

### Table ID

```hcl
output "table_id" {
  description = "ID of the DynamoDB table"
  value       = aws_dynamodb_table.employee.id
}
```

The parent module can then access:

```hcl
module.dynamodb.table_name
module.dynamodb.table_arn
module.dynamodb.table_id
```

---

## 12. Passing Module Outputs to Another Module

This is one of the most important concepts in this sprint.

Suppose the DynamoDB module provides:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

The IAM module requires the table ARN:

```hcl
variable "table_arn" {
  description = "ARN of the DynamoDB table"
  type        = string
}
```

The root module connects them:

```hcl
module "iam" {
  source = "../../modules/iam"

  table_arn = module.dynamodb.table_arn
}
```

The data flow becomes:
<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
┌──────────────────┐
│ DynamoDB Module  │
│                  │
│ creates table    │
└────────┬─────────┘
         │
         │ table_arn
         ▼
┌──────────────────┐
│   Root Module    │
└────────┬─────────┘
         │
         │ table_arn
         ▼
┌──────────────────┐
│    IAM Module    │
│                  │
│ creates policy   │
└──────────────────┘
</pre>

This is called **module composition**.

---

## 13. Module Output Creates Dependencies

When one module's output is used as another module's input, Terraform can automatically understand the dependency.

For example:

```hcl
module "lambda" {
  source = "../../modules/lambda"

  role_arn = module.iam.role_arn
}
```

Terraform understands that the Lambda module depends on the IAM module.

The relationship is:

```text
IAM Module
    │
    │ role_arn
    ▼
Lambda Module
```

This is an example of an **implicit dependency**.

No `depends_on` is required simply because one module consumes another module's output.

---

## 14. Complete Example

Consider the following DynamoDB module.

### `modules/dynamodb/variables.tf`

```hcl
variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
}

variable "hash_key" {
  description = "Partition key for the DynamoDB table"
  type        = string
}
```

### `modules/dynamodb/main.tf`

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = var.hash_key

  attribute {
    name = var.hash_key
    type = "S"
  }
}
```

### `modules/dynamodb/outputs.tf`

```hcl
output "table_name" {
  description = "Name of the DynamoDB table"
  value       = aws_dynamodb_table.employee.name
}

output "table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.employee.arn
}
```

The root module can then use:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}
```

Another module can consume the output:

```hcl
module "iam" {
  source = "../../modules/iam"

  table_arn = module.dynamodb.table_arn
}
```

The complete flow is:

```text
Root Module
    │
    │ table_name
    │ hash_key
    ▼
DynamoDB Module
    │
    │ creates table
    ▼
DynamoDB
    │
    │ table_arn
    ▼
IAM Module
    │
    │ creates policy
    ▼
Lambda Access
```

---

## 15. Root Module Outputs

Outputs do not have to stop inside the module hierarchy.

The root module can expose child module outputs.

For example:

```hcl
output "dynamodb_table_name" {
  description = "Employee DynamoDB table name"
  value       = module.dynamodb.table_name
}
```

After applying the configuration, Terraform can display the output:

```bash
terraform output
```

A specific output can also be queried:

```bash
terraform output dynamodb_table_name
```

This is useful for exposing important infrastructure information after deployment.

---

## 16. Sensitive Outputs

Terraform outputs can contain sensitive information.

For example:

```hcl
output "some_secret" {
  value     = var.some_secret
  sensitive = true
}
```

The `sensitive = true` setting tells Terraform to hide the value from normal CLI output.

Example:

```hcl
output "database_password" {
  value     = var.database_password
  sensitive = true
}
```

Sensitive values should still be handled carefully because marking an output as sensitive does not mean the value does not exist in Terraform state.

Secrets should therefore be managed using appropriate secret-management mechanisms rather than casually storing them in configuration.

---

## 17. Input and Output Design

A good module should expose only the inputs and outputs that are actually needed.

Avoid creating unnecessary variables:

```hcl
variable "resource_name"
variable "resource_type"
variable "random_setting"
variable "unused_setting"
```

Instead, keep the module interface focused.

For example:

```text
DynamoDB Module
│
├── Inputs
│   ├── table_name
│   └── hash_key
│
└── Outputs
    ├── table_name
    └── table_arn
```

A clean module interface makes the module easier to reuse.

---

## 18. Module Interface

Inputs and outputs can be thought of as a module's API.

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                 MODULE
        ┌─────────────────────┐
        │                     │
Inputs ──►   Terraform Logic  ──► Outputs
        │                     │
        └─────────────────────┘
</pre>

For example:

```text
Input:
table_name = employee-dev-table

        ↓

DynamoDB Module

        ↓

Output:
table_arn
```

This concept becomes especially important when designing production-quality modules.

---

## 19. Testing Module Changes

Whenever module inputs or outputs are modified, validate the configuration.

Run:

```bash
terraform fmt -recursive
```

Then:

```bash
terraform validate
```

Then:

```bash
terraform plan
```

If the module structure is changed significantly, initialize Terraform again:

```bash
terraform init
```

A typical workflow is:

```bash
terraform fmt -recursive
terraform init
terraform validate
terraform plan
```

---

## 20. Learned vs Implemented

### 📚 Learned

The following concepts were learned:

* Terraform module inputs
* Terraform module outputs
* Variable types
* Required variables
* Default values
* Variable descriptions
* Variable validation
* Module output references
* Passing outputs between modules
* Module composition
* Module-generated dependencies
* Root module outputs
* Sensitive outputs
* Designing clean module interfaces

### 🛠️ Implemented

The Employee Management API uses module variables to avoid hardcoding infrastructure-specific values.

For example, the DynamoDB module can receive values such as:

```text
table_name
hash_key
```

The module uses those values to create the DynamoDB resource.

The module can expose important resource information such as:

```text
table_name
table_arn
```

These outputs can then be consumed by other infrastructure components.

The resulting architecture follows:

```text
Environment
     │
     │ Inputs
     ▼
DynamoDB Module
     │
     │ Outputs
     ▼
IAM / Other Modules
```

### 🔮 Future Improvements

As the project evolves, module interfaces can be improved by adding:

* stronger variable validation
* environment-specific variable files
* standardized tagging inputs
* additional module outputs
* sensitive-value handling
* more reusable module interfaces

The module interfaces should remain intentionally small and focused.

---

## 21. Key Takeaways

> 1. Module inputs allow parent configurations to provide values to child modules.
> 2. Input variables make modules reusable.
> 3. Variables should have meaningful descriptions and explicit types.
> 4. Variables can have defaults and validation rules.
> 5. Module outputs expose useful information from child modules.
> 6. Outputs can be passed into other modules.
> 7. Module-to-module references automatically create dependencies.
> 8. Root modules can expose child module outputs.
> 9. Sensitive outputs should be handled carefully.
> 10. A module's inputs and outputs form its public interface.
> 11. Good module design keeps the interface simple and focused.

The next sprint will build on this concept by using these modules and interfaces to create **reusable infrastructure**, reducing duplication and making the Employee Management API easier to scale across environments.
