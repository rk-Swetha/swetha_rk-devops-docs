# Terraform Variables

## 1. Overview

Terraform variables allow configurations to accept values from outside the Terraform code instead of hardcoding every value directly inside resource blocks.

Variables are one of the main mechanisms used to make Terraform configurations:

* reusable
* configurable
* environment-aware
* easier to maintain
* easier to share across modules

For example, instead of hardcoding:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

we can define:

```hcl
variable "environment" {
  type = string
}
```

and use:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-${var.environment}-table"
}
```

The same Terraform configuration can then be used with:

```text
dev
test
staging
prod
```

without changing the resource definition itself.

---

# 2. What Is a Terraform Variable?

A Terraform variable is an input value that can be supplied to a Terraform configuration.

The general syntax is:

```hcl
variable "name" {
  type    = string
  default = "example"
}
```

The variable can then be referenced using:

```hcl
var.name
```

Example:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

resource "aws_dynamodb_table" "employee" {
  name = "employee-${var.environment}-table"
}
```

Terraform evaluates:

```text
var.environment
       ↓
      dev
       ↓
employee-dev-table
```

---

# 3. Why Use Variables?

Without variables, Terraform configurations often contain hardcoded values.

Example:

```hcl
resource "aws_lambda_function" "employee_api" {
  function_name = "employee-api-dev"
  runtime       = "python3.13"
}
```

This makes the configuration tightly coupled to one environment.

Variables allow the configuration to become reusable:

```hcl
variable "environment" {
  type = string
}

variable "lambda_runtime" {
  type = string
}

resource "aws_lambda_function" "employee_api" {
  function_name = "employee-api-${var.environment}"
  runtime       = var.lambda_runtime
}
```

Now different configurations can provide different values.

---

# 4. Variable Declaration

Variables are normally declared using a `variable` block.

Example:

```hcl
variable "environment" {
  type = string
}
```

The variable has:

* a name
* an optional type
* an optional default
* optional description
* optional validation
* optional sensitivity configuration
* optional nullability configuration

Example:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"
}
```

---

# 5. Variable Name

The variable name identifies the input.

Example:

```hcl
variable "environment" {
  type = string
}
```

The variable is referenced as:

```hcl
var.environment
```

Common naming examples:

```hcl
variable "project_name" {}

variable "environment" {}

variable "region" {}

variable "lambda_runtime" {}

variable "table_name" {}
```

Variable names should clearly communicate what value they represent.

---

# 6. Variable Types

Terraform supports several important variable types.

Common types include:

* `string`
* `number`
* `bool`
* `list`
* `set`
* `map`
* `object`
* `tuple`

Types help Terraform understand what kind of value is expected.

---

# 7. String Variables

A string contains text.

Example:

```hcl
variable "environment" {
  type = string
}
```

Value:

```hcl
environment = "dev"
```

Usage:

```hcl
name = "employee-${var.environment}-table"
```

Result:

```text
employee-dev-table
```

Common uses:

* environment names
* resource names
* AWS regions
* runtime names
* project names
* tags

---

# 8. Number Variables

A number represents a numeric value.

Example:

```hcl
variable "lambda_memory" {
  type    = number
  default = 512
}
```

Usage:

```hcl
memory_size = var.lambda_memory
```

Numbers can be used for:

* memory sizes
* counts
* timeouts
* capacity values
* ports
* numeric configuration

---

# 9. Boolean Variables

A boolean contains either:

```text
true
```

or:

```text
false
```

Example:

```hcl
variable "enable_logging" {
  type    = bool
  default = true
}
```

Usage:

```hcl
enabled = var.enable_logging
```

Boolean variables are useful for feature flags and optional infrastructure behavior.

---

# 10. List Variables

A list contains ordered values.

Example:

```hcl
variable "availability_zones" {
  type = list(string)
}
```

Value:

```hcl
availability_zones = [
  "ap-south-1a",
  "ap-south-1b"
]
```

List elements maintain their order.

Example:

```hcl
variable "allowed_runtimes" {
  type    = list(string)
  default = ["python3.12", "python3.13"]
}
```

---

# 11. Set Variables

A set contains unique values where ordering is not significant.

Example:

```hcl
variable "enabled_services" {
  type = set(string)
}
```

Value:

```hcl
enabled_services = [
  "lambda",
  "dynamodb",
  "api_gateway"
]
```

Sets are useful when uniqueness matters more than ordering.

---

# 12. Map Variables

A map contains key-value pairs.

Example:

```hcl
variable "tags" {
  type = map(string)
}
```

Value:

```hcl
tags = {
  Environment = "dev"
  Project     = "employee-api"
}
```

Usage:

```hcl
tags = var.tags
```

Maps are especially useful for:

* tags
* configuration values
* environment-specific settings

---

# 13. Object Variables

Objects allow structured input with named attributes.

Example:

```hcl
variable "lambda_config" {
  type = object({
    runtime = string
    memory  = number
    timeout = number
  })
}
```

Value:

```hcl
lambda_config = {
  runtime = "python3.13"
  memory  = 512
  timeout = 30
}
```

Usage:

```hcl
runtime = var.lambda_config.runtime
memory  = var.lambda_config.memory
timeout = var.lambda_config.timeout
```

Objects are useful when several related configuration values belong together.

---

# 14. Tuple Variables

A tuple is an ordered collection where each position can have a different type.

Example:

```hcl
variable "application_settings" {
  type = tuple([
    string,
    number,
    bool
  ])
}
```

Value:

```hcl
application_settings = [
  "employee-api",
  512,
  true
]
```

Tuples are less common in simple Terraform configurations but are useful when a fixed structure with different types is required.

---

# 15. Required Variables

A variable without a `default` value is generally required.

Example:

```hcl
variable "environment" {
  type = string
}
```

If Terraform does not receive a value for the variable, it asks for the value interactively or reports an error depending on how Terraform is being executed.

This is useful when a value must always be explicitly supplied.

---

# 16. Default Values

A variable can have a default value.

Example:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}
```

If no value is provided, Terraform uses:

```text
dev
```

Defaults are useful for:

* sensible development defaults
* optional configuration
* reducing repetitive input

However, defaults should not hide important environment-specific configuration.

---

# 17. Variable Descriptions

Descriptions explain what a variable represents.

Example:

```hcl
variable "environment" {
  description = "Deployment environment such as dev, test, staging, or prod"
  type        = string
  default     = "dev"
}
```

Descriptions improve:

* readability
* maintainability
* module usability
* documentation

A reusable module should generally provide descriptions for its input variables.

---

# 18. Passing Variable Values

Terraform variables can receive values through several mechanisms.

Common methods include:

1. default values
2. `terraform.tfvars`
3. `*.auto.tfvars`
4. `-var`
5. `-var-file`
6. environment variables using `TF_VAR_...`

Example using CLI:

```powershell
terraform plan -var="environment=dev"
```

Example using a variable file:

```powershell
terraform plan -var-file="dev.tfvars"
```

---

# 19. terraform.tfvars

Terraform automatically loads a file named:

```text
terraform.tfvars
```

Example:

```hcl
environment = "dev"
region      = "ap-south-1"
```

Terraform automatically uses these values when running commands such as:

```powershell
terraform plan
```

or:

```powershell
terraform apply
```

---

# 20. Auto-Loaded Variable Files

Terraform also automatically loads files ending with:

```text
.auto.tfvars
```

Example:

```text
dev.auto.tfvars
```

Example contents:

```hcl
environment = "dev"
```

This can be useful when configuration should be automatically loaded without explicitly specifying `-var-file`.

---

# 21. Custom Variable Files

Terraform also supports custom variable files.

Example:

```text
dev.tfvars
prod.tfvars
```

These are not automatically loaded simply because they end in `.tfvars`.

They can be explicitly supplied:

```powershell
terraform plan -var-file="dev.tfvars"
```

For production:

```powershell
terraform plan -var-file="prod.tfvars"
```

This is useful for environment-specific configuration.

---

# 22. Command-Line Variables

A variable can be supplied directly using:

```powershell
terraform plan -var="environment=dev"
```

Another example:

```powershell
terraform apply -var="lambda_memory=512"
```

CLI variables are useful for:

* temporary values
* automation
* testing
* CI/CD pipelines

However, using many CLI variables manually can become difficult to maintain.

---

# 23. Environment Variables

Terraform supports environment variables using the prefix:

```text
TF_VAR_
```

For example:

```text
TF_VAR_environment=dev
```

Terraform interprets this as the value for:

```hcl
variable "environment" {}
```

Environment variables are particularly useful in CI/CD pipelines.

Example:

```text
TF_VAR_environment
TF_VAR_region
TF_VAR_project_name
```

---

# 24. Variable Precedence

When the same variable receives values from multiple sources, Terraform follows a precedence order.

A simplified understanding is:

```text
Default
   ↓
Automatically loaded variable files
   ↓
Explicit variable files
   ↓
Environment variables
   ↓
CLI -var / -var-file values
```

The more specific value overrides a lower-precedence value.

When working with multiple variable sources, always verify which value Terraform is actually using.

---

# 25. Variable References

Variables are referenced using:

```hcl
var.<variable_name>
```

Example:

```hcl
variable "environment" {
  type = string
}

resource "aws_dynamodb_table" "employee" {
  name = "employee-${var.environment}-table"
}
```

The important syntax is:

```hcl
var.environment
```

---

# 26. Variables and Expressions

Variables can be used inside Terraform expressions.

Example:

```hcl
name = "${var.project_name}-${var.environment}-table"
```

Modern Terraform also supports direct interpolation within strings:

```hcl
name = "${var.project_name}-${var.environment}-table"
```

Variables can also participate in expressions:

```hcl
timeout = var.environment == "prod" ? 60 : 30
```

This allows configuration behavior to depend on input values.

---

# 27. Variable Validation

Terraform allows validation rules to restrict acceptable values.

Example:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string

  validation {
    condition     = contains(["dev", "test", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, test, staging, or prod."
  }
}
```

Now invalid input such as:

```text
environment = "development"
```

can be rejected during Terraform validation.

Variable validation helps catch configuration errors early.

---

# 28. Sensitive Variables

Some variables may contain sensitive information.

Example:

```hcl
variable "database_password" {
  type      = string
  sensitive = true
}
```

Sensitive variables are treated specially when Terraform displays values.

However, `sensitive = true` does **not** mean the value is automatically encrypted everywhere or safe to commit to Git.

Never commit secrets directly into:

```text
*.tf
*.tfvars
```

when those files are tracked by Git.

Use appropriate secret-management mechanisms for real credentials.

---

# 29. Nullable Variables

Terraform variables can control whether `null` is an accepted value.

Example:

```hcl
variable "description" {
  type     = string
  nullable = false
}
```

With:

```hcl
nullable = false
```

Terraform requires a non-null value.

Nullable behavior is useful when designing strict module interfaces.

---

# 30. Variables in Modules

Variables are especially important in reusable modules.

A child module can declare:

```hcl
variable "table_name" {
  type = string
}
```

The root module can provide:

```hcl
module "dynamodb" {
  source = "./modules/dynamodb"

  table_name = "employee-dev-table"
}
```

The data flow becomes:

```text
Root Module
    │
    │ table_name
    ▼
DynamoDB Module
    │
    ▼
DynamoDB Resource
```

This is one of the most important uses of Terraform variables.

---

# 31. Root Module vs Child Module Variables

Variables can exist in both root and child modules.

### Root module

The root module receives values from:

* `.tfvars`
* CLI
* environment variables
* automation

Example:

```hcl
variable "environment" {
  type = string
}
```

### Child module

The child module defines the inputs required by the module:

```hcl
variable "table_name" {
  type = string
}
```

The root module passes values into the child module.

```text
User / CI
   │
   ▼
Root Variables
   │
   ▼
Module Inputs
   │
   ▼
Resources
```

---

# 32. Variables in the Employee Management API

The Employee Management API uses Terraform modules for infrastructure components such as:

* DynamoDB
* IAM
* Lambda

Variables allow the same module design to receive environment-specific configuration.

For example:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}
```

A DynamoDB module could receive:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-${var.environment}-table"
  hash_key   = "employee_id"
}
```

For the current development environment:

```text
environment = dev
```

The resulting table name becomes:

```text
employee-dev-table
```

---

# 33. Project Variable Flow

The project can be understood as:

```text
Environment Configuration
          │
          ▼
      Variables
          │
          ▼
    Root Module
          │
          ├──────────────┐
          ▼              ▼
    DynamoDB Module   IAM Module
          │              │
          ▼              ▼
       Table          IAM Role
                         │
                         ▼
                   Lambda Module
```

Variables provide the configuration inputs that move through this architecture.

---

# 34. Variables vs Locals

Variables and locals serve different purposes.

### Variables

Variables are inputs supplied from outside the module.

```hcl
variable "environment" {
  type = string
}
```

Reference:

```hcl
var.environment
```

### Locals

Locals are calculated or reusable values defined inside the configuration.

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}
```

Reference:

```hcl
local.name_prefix
```

The distinction is:

```text
Variable = input
Local    = calculated/internal value
```

---

# 35. Variables vs Hardcoded Values

Hardcoded:

```hcl
name = "employee-dev-table"
```

Variable-based:

```hcl
name = "employee-${var.environment}-table"
```

Variable-based configuration is generally more reusable.

However, not every value needs to become a variable.

For example:

```hcl
hash_key = "employee_id"
```

may remain fixed if it is part of the application's design rather than environment-specific configuration.

Avoid creating variables for every single string in a Terraform configuration.

---

# 36. Variables and Environment Separation

Variables work closely with environment separation.

Example:

```text
dev.tfvars
prod.tfvars
```

Development:

```hcl
environment = "dev"
region      = "ap-south-1"
```

Production:

```hcl
environment = "prod"
region      = "ap-south-1"
```

The same Terraform configuration can then consume different values.

Conceptually:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
             Same Terraform Code
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      dev.tfvars          prod.tfvars
          │                   │
          ▼                   ▼
        DEV                 PROD
</pre>

This supports reusable infrastructure while keeping configuration separate.

---

# 37. Variables and Terraform Plan

Variables affect Terraform's plan because Terraform uses their values when evaluating configuration.

Example:

```hcl
variable "environment" {
  type = string
}

resource "aws_dynamodb_table" "employee" {
  name = "employee-${var.environment}-table"
}
```

Running:

```powershell
terraform plan -var="environment=dev"
```

can produce a plan containing:

```text
employee-dev-table
```

Changing the variable:

```powershell
terraform plan -var="environment=test"
```

can produce:

```text
employee-test-table
```

Therefore, variable values can directly affect which infrastructure Terraform proposes to create or modify.

---

# 38. Variables and Terraform State

Variable values can influence resources represented in Terraform state.

For example:

```text
var.environment
       │
       ▼
employee-dev-table
       │
       ▼
Terraform resource
       │
       ▼
State
```

State management is covered in more detail in the Terraform state and remote backend documentation.

The important concept here is that changing variable values can change the desired infrastructure configuration.

---

# 39. Variables in CI/CD

Variables are very useful in automated pipelines.

A CI/CD pipeline can provide:

```text
TF_VAR_environment=dev
```

or use:

```powershell
terraform plan -var-file="dev.tfvars"
```

This allows the same Terraform configuration to be used by automation without modifying the source code.

A common conceptual flow is:

```text
GitHub Actions
      │
      ▼
Environment Variables / tfvars
      │
      ▼
Terraform
      │
      ▼
terraform plan
      │
      ▼
terraform apply
```

---

# 40. Common Variable Mistakes

## Mistake 1 — Hardcoding everything

```hcl
name = "employee-dev-table"
```

Problem:

The configuration is difficult to reuse.

---

## Mistake 2 — Creating unnecessary variables

Example:

```hcl
variable "hash_key" {}
```

when the application permanently requires:

```text
employee_id
```

Problem:

The configuration becomes unnecessarily complicated.

---

## Mistake 3 — Missing variable type

Example:

```hcl
variable "environment" {}
```

Terraform can infer values in many situations, but explicitly declaring the expected type improves clarity and validation.

Prefer:

```hcl
variable "environment" {
  type = string
}
```

---

## Mistake 4 — Weak validation

Allowing:

```text
environment = "anything"
```

when only a known set of environments is valid can lead to configuration errors.

Use validation where appropriate.

---

## Mistake 5 — Committing secrets

Never commit real credentials such as:

```text
password
access_key
secret_key
token
```

inside tracked Terraform variable files.

---

## Mistake 6 — Making every value configurable

Turning every constant into a variable can make Terraform harder to understand.

Only expose values that genuinely need to be configurable.

---

# 41. Variable Best Practices

### 1. Use meaningful names

Prefer:

```hcl
variable "environment" {}
```

over:

```hcl
variable "env1" {}
```

### 2. Define appropriate types

Example:

```hcl
variable "lambda_memory" {
  type = number
}
```

### 3. Add descriptions

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
}
```

### 4. Use validation for constrained inputs

```hcl
validation {
  condition     = contains(["dev", "test", "prod"], var.environment)
  error_message = "Invalid environment."
}
```

### 5. Avoid hardcoded environment values

Prefer:

```hcl
"employee-${var.environment}-table"
```

over:

```hcl
"employee-dev-table"
```

when the resource is intended to be reusable.

### 6. Protect sensitive values

Use:

```hcl
sensitive = true
```

for sensitive inputs and use proper secret-management practices.

### 7. Keep module interfaces simple

Expose only the inputs that a module actually needs.

---

# 42. Practical Variable Workflow

A typical workflow is:

```text
1. Identify configurable value
        │
        ▼
2. Declare variable
        │
        ▼
3. Define type
        │
        ▼
4. Add description/default if appropriate
        │
        ▼
5. Add validation if required
        │
        ▼
6. Reference using var.<name>
        │
        ▼
7. Supply value using tfvars / CLI / environment
        │
        ▼
8. Run terraform fmt
        │
        ▼
9. Run terraform validate
        │
        ▼
10. Run terraform plan
```

---

# 43. Practical Example

Variable declaration:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "test", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, test, staging, or prod."
  }
}
```

Resource:

```hcl
resource "aws_dynamodb_table" "employee" {
  name     = "employee-${var.environment}-table"
  hash_key = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}
```

The same resource definition can support:

```text
employee-dev-table
employee-test-table
employee-staging-table
employee-prod-table
```

depending on the variable value.

---

# 44. Variables and the Project's Current Implementation

The current project uses Terraform modules and environment-oriented configuration.

Variables are therefore important for making those modules reusable.

The implementation principle is:

```text
Environment-specific value
          │
          ▼
       Variable
          │
          ▼
      Module Input
          │
          ▼
       Resource
```

For example:

```text
environment = dev
      │
      ▼
var.environment
      │
      ▼
employee-dev-table
```

This avoids embedding environment-specific configuration directly into reusable module logic.

---

# 45. Learned vs Implemented

## 📚 Learned

The following concepts were covered:

* Terraform input variables
* variable declarations
* variable names
* variable types
* strings
* numbers
* booleans
* lists
* sets
* maps
* objects
* tuples
* required variables
* default values
* variable descriptions
* `.tfvars`
* `terraform.tfvars`
* `.auto.tfvars`
* `-var`
* `-var-file`
* `TF_VAR_*`
* variable precedence
* variable references
* expressions
* validation
* sensitive variables
* nullable variables
* root module variables
* child module variables
* variables vs locals
* variables and environment separation
* variables in CI/CD

## 🛠️ Implemented

The project already uses Terraform configuration that is designed around reusable modules and environment-specific values.

Variables support this architecture by allowing values such as:

```text
environment
table names
Lambda configuration
module inputs
```

to be supplied rather than embedding every value directly into reusable infrastructure logic.

The current project remains focused on the development environment, so documentation should not claim that full production environment configuration has already been implemented.

## 🔮 Future Improvements

Potential future improvements include:

* stronger variable validation
* dedicated `dev.tfvars`, `test.tfvars`, and `prod.tfvars`
* CI/CD-driven variable injection
* improved secret management
* environment-specific configuration
* stricter production variable validation
* standardized variable interfaces across modules

---

# 46. Key Takeaways

1. **Variables provide inputs to Terraform configurations.**

2. Variables are declared using:

```hcl
variable "name" {}
```

3. Variables are referenced using:

```hcl
var.name
```

4. Terraform supports multiple variable types including:

```text
string
number
bool
list
set
map
object
tuple
```

5. Variables can receive values through:

```text
defaults
terraform.tfvars
*.auto.tfvars
-var
-var-file
TF_VAR_*
```

6. Validation can prevent invalid configuration values.

7. Sensitive variables should not be treated as a replacement for proper secret management.

8. Variables are especially important for reusable modules.

9. Variables represent **inputs**, while locals represent **internal calculated values**.

10. Avoid both extremes:

```text
Too many hardcoded values
        ❌
        │
        ▼
Too many unnecessary variables
        ❌
```

Aim for:

```text
Meaningful configurable inputs
            │
            ▼
      Reusable Terraform
```

### Core Principle

> **Use variables for values that should be configurable, reusable, or environment-specific, while keeping true constants inside the Terraform configuration.**
