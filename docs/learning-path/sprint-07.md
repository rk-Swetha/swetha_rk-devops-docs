# Sprint 7 — Terraform Modules

## Overview

As Terraform projects grow, keeping every resource inside a single `main.tf` becomes difficult to maintain.

For example, the Employee Management API contains multiple AWS resources:

```text
Employee Management API
│
├── DynamoDB
├── IAM
├── Lambda
├── API Gateway
└── CloudWatch
```

If all of these resources are written together in one Terraform configuration, the project can quickly become difficult to understand and modify.

Terraform **modules** provide a way to organize related infrastructure into reusable components.

A module can contain:

* Resources
* Variables
* Outputs
* Providers
* Supporting Terraform configuration

The goal is to make infrastructure easier to:

* organize
* reuse
* maintain
* test
* understand
* scale

---

## 1. What is a Terraform Module?

A Terraform module is a collection of Terraform configuration files that are grouped together for a specific purpose.

For example:

```text
modules/
│
├── dynamodb/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── iam/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
└── lambda/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

Each directory represents a module.

Instead of defining every resource directly in the root configuration, the root module can call these child modules.

---

## 2. Root Module vs Child Module

Terraform uses the concept of a **root module** and **child modules**.

### Root Module

The directory from which Terraform commands are executed is the root module.

For example:

```text
terraform/
└── environments/
    └── dev/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

If Terraform is executed from `environments/dev`:

```bash
terraform init
terraform plan
terraform apply
```

then `environments/dev` acts as the root module.

### Child Module

A module called by another Terraform configuration is a child module.

For example:

```text
modules/
└── dynamodb/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

The root module can call it using:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}
```

The relationship becomes:

```text
Root Module
     │
     ├── DynamoDB Module
     ├── IAM Module
     └── Lambda Module
```

---

## 3. Why Use Modules?

Without modules, Terraform configurations can become large and difficult to maintain.

For example:

```text
main.tf
│
├── DynamoDB resources
├── IAM resources
├── Lambda resources
├── API Gateway resources
├── CloudWatch resources
└── Security resources
```

With modules:

```text
modules/
│
├── dynamodb/
├── iam/
├── lambda/
├── api_gateway/
└── cloudwatch/
```

The infrastructure becomes easier to understand.

The root module becomes responsible for **composing the infrastructure**, while individual modules are responsible for specific infrastructure components.

---

## 4. Module Directory Structure

A commonly used module structure is:

```text
modules/
└── dynamodb/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

### `main.tf`

Contains the resources created by the module.

Example:

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

### `variables.tf`

Defines the inputs accepted by the module.

Example:

```hcl
variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
}

variable "hash_key" {
  description = "DynamoDB partition key"
  type        = string
}
```

### `outputs.tf`

Defines values that the module makes available to the parent module.

Example:

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

---

## 5. Calling a Module

A module is called using a `module` block.

Example:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}
```

The `source` tells Terraform where the module is located.

The other arguments provide values to the module's variables.

The flow is:

```text
Root Module
     │
     │ module "dynamodb"
     ▼
DynamoDB Module
     │
     ├── variables.tf
     ├── main.tf
     └── outputs.tf
```

---

## 6. Local Modules

The Employee Management API uses local Terraform modules.

A local module can be referenced using a relative path.

Example:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}
```

The `source` path depends on the directory structure of the project.

For example:

```text
terraform/
│
├── environments/
│   └── dev/
│       └── main.tf
│
└── modules/
    └── dynamodb/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

From `environments/dev`, the module is two levels above:

```text
../../modules/dynamodb
```

---

## 7. Module Inputs

Modules should avoid hardcoding values wherever possible.

For example, this is less reusable:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

Instead, use a variable:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = var.table_name
}
```

The caller then provides the value:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
}
```

This allows the same module to be reused.

For example:

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

The module itself does not need to change.

---

## 8. Module Outputs

Modules can expose useful resource information through outputs.

For example:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

The parent module can access it using:

```hcl
module.dynamodb.table_arn
```

For example:

```hcl
resource "aws_iam_policy" "lambda_dynamodb" {
  name = "lambda-dynamodb-policy"

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "dynamodb:GetItem",
        "dynamodb:PutItem"
      ]

      Resource = module.dynamodb.table_arn
    }]
  })
}
```

This creates a clean connection:

```text
DynamoDB Module
      │
      │ table_arn
      ▼
IAM Policy
```

---

## 9. Module Composition

Modules can be combined to build larger infrastructure.

For the Employee Management API:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                  Root Module
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      DynamoDB        IAM         Lambda
          │            │            │
          └────────────┼────────────┘
                       ▼
                  API Gateway
</pre>


Each module handles a specific infrastructure concern.

The root module brings them together.

This approach is called **module composition**.

---

## 10. Modules and Reusability

One of the biggest benefits of modules is reusability.

Consider a DynamoDB module:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}
```

The same module can be used for another application:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "orders-dev-table"
  hash_key   = "order_id"
}
```

The underlying module implementation remains the same.

Only the inputs change.

This follows the principle:

```text
Reusable Logic
      +
Different Inputs
      ↓
Different Infrastructure
```

---

## 11. Terraform Module Initialization

Whenever a configuration introduces or changes a module, Terraform needs to initialize the module.

Run:

```bash
terraform init
```

Terraform downloads or prepares the required modules.

You can then inspect the configuration using:

```bash
terraform validate
```

and preview changes using:

```bash
terraform plan
```

A typical workflow is:

```bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan
```

---

## 12. Module Dependency Flow

Modules can also participate in Terraform's dependency graph.

For example:

```text
DynamoDB Module
       │
       │ output: table_arn
       ▼
IAM Module
       │
       │ output: role_arn
       ▼
Lambda Module
```

If one module consumes an output from another module, Terraform can determine the dependency automatically.

Example:

```hcl
module "lambda" {
  source = "../../modules/lambda"

  role_arn = module.iam.role_arn
}
```

Terraform understands:

```text
IAM Module
     ↓
Lambda Module
```

because the Lambda module references the IAM module's output.

---

## 13. Module Best Practices

Terraform modules should be designed carefully.

### Keep modules focused

A module should generally represent a logical infrastructure component.

Good:

```text
modules/
├── dynamodb/
├── iam/
└── lambda/
```

Less desirable:

```text
modules/
└── everything/
```

### Avoid unnecessary hardcoding

Prefer:

```hcl
name = var.table_name
```

over:

```hcl
name = "employee-dev-table"
```

### Use meaningful variable names

Prefer:

```hcl
variable "table_name"
```

instead of:

```hcl
variable "x"
```

### Document variables and outputs

Descriptions make modules easier for other developers to understand.

### Avoid excessive module fragmentation

Not every single resource needs to become its own module.

The goal is **logical reuse**, not creating hundreds of tiny modules.

---

## 14. Modules in the Employee Management API

The Employee Management API can be organized into reusable modules such as:

```text
terraform/
│
├── modules/
│   │
│   ├── dynamodb/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   ├── iam/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   └── lambda/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
└── environments/
    └── dev/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

The environment configuration becomes responsible for combining these modules.

Example:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = var.table_name
  hash_key   = "employee_id"
}
```

Then:

```hcl
module "lambda" {
  source = "../../modules/lambda"

  function_name = var.function_name
  role_arn      = module.iam.role_arn
}
```

The exact module structure can evolve as additional AWS services are introduced.

---

## 15. Learned vs Implemented

### 📚 Learned

The following concepts were learned:

* What Terraform modules are
* Root modules
* Child modules
* Local modules
* Module directory structure
* Module inputs
* Module outputs
* Module composition
* Module reusability
* Module dependencies
* Module initialization
* Terraform module best practices

### 🛠️ Implemented

The Employee Management API Terraform project is designed around reusable infrastructure components.

The existing project structure uses logical Terraform modules for infrastructure components such as:

```text
DynamoDB
IAM
Lambda
```

The root/environment configuration can consume these modules instead of duplicating resource definitions.

Module inputs are used to provide environment-specific values such as:

```text
employee-dev-table
employee-api-dev
```

while the module implementation remains reusable.

### 🔮 Future Improvements

As the project grows, additional infrastructure can be separated into modules where there is a clear reuse or maintenance benefit.

Potential modules include:

```text
API Gateway
CloudWatch
KMS
SNS
```

The project can eventually follow:

```text
Reusable Modules
       ↓
Environment Configuration
       ↓
AWS Infrastructure
```

This will become especially useful when multiple environments such as `dev`, `staging`, and `prod` are introduced.

---

## 16. Key Takeaways

> 1. Terraform modules group related infrastructure into reusable components.
> 2. The root module composes the infrastructure.
> 3. Child modules contain reusable infrastructure logic.
> 4. `variables.tf` defines module inputs.
> 5. `outputs.tf` exposes useful values from a module.
> 6. Local modules can be referenced using a relative `source` path.
> 7. Module outputs can create dependencies between modules.
> 8. Modules reduce duplication and improve maintainability.
> 9. Modules should represent logical infrastructure components rather than every individual resource.
> 10. Reusable modules become especially valuable when supporting multiple environments.

Terraform modules are the foundation for building **clean, scalable, and reusable Infrastructure as Code**. The next sprint will build directly on this concept by going deeper into **module inputs and outputs**, including how values flow between the root module and child modules.
