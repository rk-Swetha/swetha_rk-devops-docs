# Sprint 9 — Reusable Infrastructure

## Overview

One of the major advantages of Terraform is the ability to define infrastructure as reusable code.

As infrastructure grows, copying and modifying the same Terraform configuration for every environment or application creates unnecessary duplication.

For example, imagine creating the same DynamoDB infrastructure separately for:

```text
Development
Testing
Production
```

Without reusable infrastructure, the project could contain:

```text
dev/main.tf
test/main.tf
prod/main.tf
```

with nearly identical resource definitions.

This creates several problems:

* duplicated code
* inconsistent configurations
* difficult maintenance
* increased chance of configuration drift
* more effort when infrastructure changes

Terraform modules help solve this problem.

The goal is:

```text
Reusable Infrastructure Logic
            │
            ├── dev
            ├── test
            └── prod
```

The infrastructure logic remains reusable while environment-specific values are supplied externally.

---

## 1. What is Reusable Infrastructure?

Reusable infrastructure means creating Terraform configurations that can be used multiple times with different inputs.

For example:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}
```

The same module can be used elsewhere:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-prod-table"
  hash_key   = "employee_id"
}
```

The module implementation does not change.

Only the inputs change.

The concept is:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
             Reusable Module
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         dev       test      prod
          │         │         │
          ▼         ▼         ▼
           Different values
</pre>

---

## 2. Why Reusability Matters

Consider a project with three environments.

Without modules:

```text
dev/
├── dynamodb resources
├── IAM resources
└── Lambda resources

test/
├── dynamodb resources
├── IAM resources
└── Lambda resources

prod/
├── dynamodb resources
├── IAM resources
└── Lambda resources
```

A change to the Lambda configuration may need to be repeated three times.

With modules:

```text
modules/
├── dynamodb/
├── iam/
└── lambda/

environments/
├── dev/
├── test/
└── prod/
```

Each environment consumes the same infrastructure modules.

This provides:

* less duplication
* consistent infrastructure
* easier maintenance
* easier upgrades
* easier review

---

## 3. DRY Principle

Reusable Terraform follows the **DRY principle**:

> Don't Repeat Yourself.

Instead of repeating:

```hcl
resource "aws_dynamodb_table" "employee" {
  # same configuration
}
```

in multiple places, define the infrastructure once inside a module.

Then reuse it:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}
```

This creates:

```text
One Infrastructure Definition
            │
            ├── Development
            ├── Testing
            └── Production
```

However, DRY should not be taken too far.

The goal is to remove meaningful duplication without making the Terraform configuration unnecessarily complicated.

---

## 4. Reusable Modules

A reusable module should generally have:

```text
modules/
└── dynamodb/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

The module contains the infrastructure logic.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = var.billing_mode
  hash_key     = var.hash_key

  attribute {
    name = var.hash_key
    type = "S"
  }
}
```

The values are supplied through variables.

This prevents the module from being tied to one specific environment.

---

## 5. Environment-Specific Inputs

A reusable module should not normally contain environment-specific names such as:

```hcl
name = "employee-dev-table"
```

Instead:

```hcl
name = var.table_name
```

The environment supplies:

```hcl
table_name = "employee-dev-table"
```

Another environment can supply:

```hcl
table_name = "employee-prod-table"
```

The module remains unchanged.

This separation is important:

```text
Module
│
├── Infrastructure Logic
│
└── No Environment-Specific Values

Environment
│
└── Supplies Environment-Specific Values
```

---

## 6. Reusable Infrastructure Architecture

A scalable Terraform project can follow a structure such as:

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
    │
    ├── dev/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars
    │
    ├── test/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars
    │
    └── prod/
        ├── main.tf
        ├── variables.tf
        └── terraform.tfvars
```

This structure separates:

```text
Reusable Logic
      from
Environment Configuration
```

The environment structure will be explored in greater detail in Sprint 10.

---

## 7. Reusing the Same Module

Suppose the DynamoDB module accepts:

```hcl
variable "table_name" {
  type = string
}

variable "hash_key" {
  type = string
}
```

Development can use:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}
```

Production can use:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-prod-table"
  hash_key   = "employee_id"
}
```

The module itself is identical.

Only the inputs differ.

---

## 8. Reusing Modules for Multiple Resources

A module can also be instantiated multiple times.

For example, suppose the project needs two DynamoDB tables:

```hcl
module "employee_table" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}

module "audit_table" {
  source = "../../modules/dynamodb"

  table_name = "employee-audit-dev-table"
  hash_key   = "audit_id"
}
```

The same module is reused twice.

Terraform treats these as separate module instances:

```text
DynamoDB Module
      │
      ├── employee_table
      │
      └── audit_table
```

This demonstrates that modules are not limited to one resource instance.

---

## 9. Module Naming

Module names should clearly describe their purpose.

Good:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}
```

Good:

```hcl
module "employee_table" {
  source = "../../modules/dynamodb"
}
```

Less useful:

```hcl
module "module1" {
  source = "../../modules/dynamodb"
}
```

Clear names make the root configuration easier to understand.

For example:

```hcl
module "iam" {
  source = "../../modules/iam"
}

module "lambda" {
  source = "../../modules/lambda"
}
```

is immediately understandable.

---

## 10. Standardizing Tags

Reusable infrastructure can also help standardize resource tags.

For example, a module could accept:

```hcl
variable "tags" {
  description = "Tags applied to resources"
  type        = map(string)
  default     = {}
}
```

The resource can then use:

```hcl
tags = var.tags
```

The environment can provide:

```hcl
tags = {
  Environment = "dev"
  Project     = "employee-api"
  ManagedBy   = "terraform"
}
```

This allows common metadata to be consistently applied.

A reusable tagging strategy becomes increasingly important as the number of AWS resources grows.

---

## 11. Reusable Naming Conventions

Naming can also be standardized.

Instead of independently constructing names everywhere:

```text
employee-dev-table
employee-dev-lambda
employee-dev-api
```

a consistent naming strategy can be used.

For example, the module may accept:

```hcl
variable "name_prefix" {
  description = "Prefix used for resource names"
  type        = string
}
```

Then:

```hcl
name = "${var.name_prefix}-table"
```

The environment could provide:

```hcl
name_prefix = "employee-dev"
```

This results in:

```text
employee-dev-table
```

A consistent naming convention makes resources easier to identify and manage.

---

## 12. Avoiding Over-Abstraction

Reusable infrastructure does not mean that everything must become a module.

For example, creating:

```text
modules/
├── string/
├── name/
├── tag/
├── attribute/
└── resource/
```

would make the project unnecessarily complicated.

A module should provide meaningful infrastructure abstraction.

Good examples include:

```text
DynamoDB Module
IAM Module
Lambda Module
API Gateway Module
```

The principle is:

```text
Meaningful Reuse
      +
Clear Abstraction
      =
Good Module Design
```

---

## 13. Module Versioning

Reusable modules eventually need version management.

For local modules:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}
```

the module changes along with the repository.

For externally published modules, Terraform supports version constraints.

Example:

```hcl
module "example" {
  source  = "some-registry/module/provider"
  version = "1.0.0"
}
```

Versioning is particularly important when multiple projects depend on the same shared module.

A future production setup may use a private module registry or another controlled module distribution mechanism.

---

## 14. Local Modules vs Registry Modules

Terraform modules can come from different sources.

### Local Module

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}
```

The module exists inside the repository.

### Terraform Registry Module

A module can also be consumed from a registry:

```hcl
module "example" {
  source  = "some-registry/module/provider"
  version = "1.0.0"
}
```

### Git Module

Modules can also be sourced from Git repositories:

```hcl
module "example" {
  source = "git::https://example.com/example/terraform-module.git"
}
```

For the Employee Management API project, local modules are appropriate because the infrastructure is being developed and controlled within the same repository.

---

## 15. Reusable Infrastructure and Team Collaboration

Reusable modules also improve collaboration.

Without modules:

```text
Developer A → modifies dev
Developer B → modifies test
Developer C → modifies prod
```

This can lead to differences between environments.

With reusable modules:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                Shared Module
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         dev        test       prod
</pre>

The infrastructure logic remains consistent.

Changes can be reviewed centrally before being used across environments.

---

## 16. Reusability in the Employee Management API

The Employee Management API follows the reusable infrastructure concept.

A simplified structure is:

```text
terraform/
│
├── modules/
│   ├── dynamodb/
│   ├── iam/
│   └── lambda/
│
└── environment configuration
```

The modules contain reusable infrastructure logic.

For example:

```text
DynamoDB Module
      │
      ├── table_name
      ├── hash_key
      └── other configuration
```

The environment provides the values.

Similarly:

```text
IAM Module
      │
      ├── role name
      ├── permissions
      └── resource references
```

and:

```text
Lambda Module
      │
      ├── function name
      ├── runtime
      ├── handler
      └── IAM role
```

This creates a foundation for supporting multiple environments without duplicating the underlying infrastructure logic.

---

## 17. Reusable Infrastructure Workflow

A typical workflow is:

```text
1. Identify repeated infrastructure
             ↓
2. Create a module
             ↓
3. Define module inputs
             ↓
4. Define module outputs
             ↓
5. Call the module
             ↓
6. Supply environment-specific values
             ↓
7. Validate
             ↓
8. Plan
```

Commands:

```bash
terraform fmt -recursive
terraform init
terraform validate
terraform plan
```

This workflow should be followed whenever reusable Terraform infrastructure is changed.

---

## 18. Reusability vs Environment Management

Reusable modules and environment management are related, but they are not exactly the same thing.

### Reusable Module

Answers:

> How do we define infrastructure once and reuse it?

Example:

```text
modules/dynamodb
```

### Environment Configuration

Answers:

> Which values and infrastructure should this environment use?

Example:

```text
environments/dev
environments/prod
```

The relationship is:

```text
Reusable Module
       │
       │ reused by
       ▼
Environment Configuration
       │
       │ supplies values
       ▼
AWS Infrastructure
```

Sprint 10 will focus specifically on environment management.

---

## 19. Learned vs Implemented

### 📚 Learned

The following concepts were learned:

* Reusable Infrastructure
* DRY principle
* Reusable Terraform modules
* Environment-specific inputs
* Multiple module instances
* Module naming
* Standardized tags
* Naming conventions
* Avoiding over-abstraction
* Local modules
* Registry modules
* Git-based modules
* Module versioning
* Reusability and team collaboration
* Difference between reusable modules and environments

### 🛠️ Implemented

The Employee Management API infrastructure uses reusable Terraform modules for logical infrastructure components.

The architecture separates reusable infrastructure logic from environment-specific configuration.

The reusable components include areas such as:

```text
DynamoDB
IAM
Lambda
```

Module inputs allow environment-specific values such as:

```text
employee-dev-table
employee-api-dev
```

to be supplied without hardcoding those values into the reusable module logic.

The project therefore follows:

```text
Reusable Terraform Modules
            │
            ▼
Environment-specific Inputs
            │
            ▼
AWS Infrastructure
```

### 🔮 Future Improvements

The reusable infrastructure architecture can be expanded by:

* adding API Gateway modules
* adding CloudWatch modules
* adding KMS modules
* adding SNS modules
* standardizing resource tags
* standardizing naming conventions
* introducing multiple environments
* introducing module versioning
* using private module registries for shared enterprise modules

The goal should remain focused on meaningful reuse rather than unnecessary abstraction.

---

## 20. Key Takeaways

> 1. Reusable infrastructure reduces Terraform code duplication.
> 2. Terraform modules are the primary mechanism for infrastructure reuse.
> 3. The DRY principle helps prevent repeated infrastructure definitions.
> 4. Environment-specific values should be supplied as module inputs.
> 5. The same module can be instantiated multiple times.
> 6. Clear module naming improves maintainability.
> 7. Standardized tags and naming conventions improve resource management.
> 8. Not every Terraform resource needs to become a module.
> 9. Local modules are useful while developing infrastructure within the same repository.
> 10. Registry and Git modules can support broader reuse.
> 11. Module versioning becomes important when modules are shared across projects.
> 12. Reusable modules and environment configuration solve different problems.
> 13. Good module design balances reuse with simplicity.

Reusable infrastructure provides the foundation for managing multiple environments consistently. The next sprint will build on this architecture by introducing **Terraform environment management**, where the same reusable modules can be configured differently for development, testing, and production.
