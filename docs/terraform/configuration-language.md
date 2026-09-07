# Terraform Configuration Language

## 1. Overview

Terraform Configuration Language is the language used to define infrastructure and Terraform configuration.

Terraform primarily uses **HashiCorp Configuration Language (HCL)**.

HCL is designed to be:

* Human-readable
* Declarative
* Structured
* Easy to review
* Suitable for infrastructure configuration

Terraform configuration files normally use the `.tf` extension.

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

Terraform reads this configuration and determines what infrastructure should exist.

---

## 2. HCL and Terraform

HCL stands for **HashiCorp Configuration Language**.

Terraform uses HCL as its primary configuration language.

The basic relationship is:

```text
Terraform Configuration
        │
        │ written using
        ▼
      HCL
        │
        ▼
     Terraform
        │
        ▼
Infrastructure
```

Terraform also supports JSON syntax for configuration files, but HCL is normally preferred because it is easier for humans to read and maintain.

Typical Terraform files include:

```text
main.tf
variables.tf
outputs.tf
provider.tf
versions.tf
```

All of these files can contain HCL configuration.

---

## 3. Terraform Configuration Structure

Terraform configuration is built using several fundamental constructs.

Common constructs include:

```text
Blocks
Arguments
Expressions
References
Functions
Variables
Conditional Expressions
For Expressions
Dynamic Blocks
```

A simplified structure looks like:

```text
Terraform Configuration
│
├── Blocks
│    ├── Resource
│    ├── Provider
│    ├── Variable
│    ├── Output
│    ├── Module
│    └── Data
│
├── Arguments
│
├── Expressions
│
├── References
│
└── Functions
```

Understanding these constructs is essential for writing maintainable Terraform configuration.

---

## 4. Blocks

A **block** is a structural element in Terraform configuration.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

Here:

```text
resource
```

is the block type.

The block also contains labels:

```text
aws_dynamodb_table
employee
```

The general structure is:

```hcl
BLOCK_TYPE "LABEL_1" "LABEL_2" {
  ARGUMENT = VALUE
}
```

Different Terraform blocks use different numbers of labels.

### Resource Block

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

### Variable Block

```hcl
variable "table_name" {
  ...
}
```

### Output Block

```hcl
output "table_name" {
  ...
}
```

### Provider Block

```hcl
provider "aws" {
  region = "us-east-1"
}
```

### Module Block

```hcl
module "dynamodb" {
  source = "../modules/dynamodb"
}
```

### Data Block

```hcl
data "aws_caller_identity" "current" {}
```

Blocks provide the overall structure of Terraform configuration.

---

## 5. Arguments

An **argument** assigns a value to a particular configuration setting.

Example:

```hcl
name = "employee-dev-table"
```

Here:

```text
name
 │
 └── Argument name

"employee-dev-table"
 │
 └── Argument value
```

Another example:

```hcl
billing_mode = "PAY_PER_REQUEST"
```

Arguments are commonly used inside Terraform blocks.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

The resource block defines the infrastructure object, while the arguments configure its properties.

---

## 6. Expressions

Terraform expressions are used to produce values.

An expression can be:

* A literal value
* A variable reference
* A resource reference
* A function call
* A conditional expression
* A collection expression
* A complex expression

### Literal Value

```hcl
name = "employee-dev-table"
```

### Variable Reference

```hcl
name = var.table_name
```

### Resource Reference

```hcl
name = aws_dynamodb_table.employee.name
```

### Function Call

```hcl
name = lower(var.table_name)
```

### Conditional Expression

```hcl
environment = var.environment == "prod" ? "production" : "non-production"
```

Expressions allow Terraform configurations to become dynamic and reusable.

---

## 7. References

Terraform uses references to access values from other parts of the configuration.

### Variable Reference

```hcl
var.table_name
```

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = var.table_name
}
```

### Resource Reference

```hcl
aws_dynamodb_table.employee.arn
```

This accesses the ARN of the DynamoDB resource.

### Data Source Reference

```hcl
data.aws_caller_identity.current.account_id
```

### Module Output Reference

```hcl
module.dynamodb.table_arn
```

These references allow information to flow between different Terraform components.

Conceptually:

```text
Variable
   │
   ▼
Resource
   │
   ▼
Resource Attribute
   │
   ▼
Module Output
```

References also help Terraform understand relationships between resources.

---

## 8. Attribute References

Terraform resources expose attributes that can be referenced elsewhere.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

The resource can be referenced as:

```hcl
aws_dynamodb_table.employee.name
```

Similarly, its ARN can be referenced as:

```hcl
aws_dynamodb_table.employee.arn
```

A common pattern is:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = var.table_name
}
```

Then another resource can reference it:

```hcl
some_argument = aws_dynamodb_table.employee.arn
```

Terraform can use this reference to understand that the second resource depends on the DynamoDB resource.

---

## 9. Strings

Strings represent text values.

Example:

```hcl
region = "us-east-1"
```

Strings can also contain variable interpolation.

```hcl
name = "employee-${var.environment}-table"
```

If:

```text
environment = "dev"
```

the resulting value becomes:

```text
employee-dev-table
```

Modern Terraform configurations can often use direct expressions instead of unnecessary interpolation.

For example:

```hcl
name = var.table_name
```

is preferred over:

```hcl
name = "${var.table_name}"
```

when the entire value is simply the variable.

---

## 10. Numbers and Booleans

Terraform supports numeric and boolean values.

### Number

```hcl
timeout = 30
```

### Boolean

```hcl
enabled = true
```

Boolean values can be:

```text
true
false
```

They are not strings.

For example:

```hcl
enabled = true
```

is different from:

```hcl
enabled = "true"
```

The first is a boolean and the second is a string.

Using the correct type helps Terraform validate configuration properly.

---

## 11. Collections

Terraform supports several collection types.

Common types include:

```text
list
set
map
tuple
object
```

### List

A list is an ordered collection.

```hcl
availability_zones = [
  "us-east-1a",
  "us-east-1b"
]
```

Values can be accessed by index:

```hcl
var.availability_zones[0]
```

### Set

A set contains unique values and does not rely on a stable ordering.

```hcl
allowed_regions = toset([
  "us-east-1",
  "us-west-2"
])
```

### Map

A map contains key-value pairs.

```hcl
tags = {
  Environment = "dev"
  Project     = "employee-api"
}
```

A value can be accessed using:

```hcl
var.tags["Environment"]
```

### Object

An object contains named attributes with defined types.

Example:

```hcl
variable "environment_config" {
  type = object({
    name   = string
    region = string
  })
}
```

Collections allow Terraform configuration to represent structured infrastructure data.

---

## 12. Comments

Terraform supports comments.

### Single-line Comment

```hcl
# This is a comment
```

Terraform also supports:

```hcl
// This is also a comment
```

### Multi-line Comment

```hcl
/*
This is a
multi-line comment.
*/
```

Comments should be used to explain configuration where the purpose is not obvious.

Avoid comments that simply repeat what the code already clearly expresses.

---

## 13. Variables and Expressions

Variables make Terraform configuration reusable.

Example:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
}
```

The variable can then be referenced:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-${var.environment}-table"
}
```

For:

```text
environment = "dev"
```

Terraform produces:

```text
employee-dev-table
```

This approach prevents environment-specific values from being hardcoded throughout the configuration.

---

## 14. Local Values

Terraform supports **local values**, which allow expressions to be assigned a reusable local name.

Example:

```hcl
locals {
  environment = "dev"

  common_tags = {
    Environment = "dev"
    Project     = "employee-api"
  }
}
```

A local value can be referenced using:

```hcl
local.environment
```

or:

```hcl
local.common_tags
```

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-${local.environment}-table"

  tags = local.common_tags
}
```

Locals are useful when a value or expression needs to be reused within a module.

They should not be used simply to give every value another name.

---

## 15. Functions

Terraform provides built-in functions for manipulating values.

Examples include:

```text
lower()
upper()
length()
join()
split()
lookup()
merge()
concat()
toset()
tolist()
```

### `lower()`

```hcl
name = lower(var.environment)
```

### `upper()`

```hcl
environment = upper(var.environment)
```

### `length()`

```hcl
count = length(var.subnets)
```

### `merge()`

```hcl
tags = merge(
  local.common_tags,
  var.additional_tags
)
```

Functions allow Terraform expressions to perform common transformations without external scripting.

---

## 16. Conditional Expressions

Conditional expressions allow Terraform to choose between two values.

The syntax is:

```text
condition ? true_value : false_value
```

Example:

```hcl
instance_type = var.environment == "prod" ? "large" : "small"
```

Conceptually:

```text
             environment == prod?
                    │
             ┌──────┴──────┐
            Yes            No
             │              │
          "large"         "small"
```

Conditional expressions are useful when infrastructure behavior needs to vary between environments.

However, complex nested conditions can make Terraform configuration difficult to understand and should be avoided when possible.

---

## 17. For Expressions

For expressions allow Terraform to transform collections.

Example:

```hcl
variable "environments" {
  type = list(string)

  default = [
    "dev",
    "test",
    "prod"
  ]
}
```

A for expression can transform the values:

```hcl
environment_names = [
  for environment in var.environments :
  upper(environment)
]
```

The result is conceptually:

```text
DEV
TEST
PROD
```

For expressions are useful for:

* Transforming collections
* Filtering values
* Creating derived collections
* Simplifying repetitive expressions

---

## 18. Dynamic Blocks

Dynamic blocks allow Terraform to generate repeated nested blocks based on a collection.

Conceptually:

```text
Collection
    │
    ▼
Dynamic Block
    │
    ├── Nested Block 1
    ├── Nested Block 2
    └── Nested Block 3
```

Example:

```hcl
dynamic "tag" {
  for_each = var.tags

  content {
    key   = tag.key
    value = tag.value
  }
}
```

Dynamic blocks are useful when a provider resource contains repeated nested configuration blocks.

However, they should not be used unnecessarily.

If normal static blocks are easier to understand, static blocks are generally preferable.

---

## 19. Meta-Arguments

Terraform provides special arguments that affect how resources are managed.

Common meta-arguments include:

```text
count
for_each
depends_on
lifecycle
provider
```

### `count`

Creates multiple instances based on a numeric count.

```hcl
resource "aws_s3_bucket" "example" {
  count = 2

  bucket = "example-${count.index}"
}
```

### `for_each`

Creates multiple instances based on a collection.

```hcl
resource "aws_s3_bucket" "example" {
  for_each = toset(["dev", "test"])

  bucket = "example-${each.key}"
}
```

### `depends_on`

Creates an explicit dependency.

```hcl
resource "example_resource" "example" {
  depends_on = [
    example_dependency.example
  ]
}
```

### `lifecycle`

Controls certain resource lifecycle behaviors.

```hcl
lifecycle {
  prevent_destroy = true
}
```

Meta-arguments are powerful and should be used intentionally.

---

## 20. Resource Addressing

Terraform uses addresses to uniquely identify resources and other objects.

A simple resource address is:

```text
aws_dynamodb_table.employee
```

The structure is:

```text
resource_type.resource_name
```

For a module:

```text
module.dynamodb
```

For a resource inside a module:

```text
module.dynamodb.aws_dynamodb_table.employee
```

When `count` or `for_each` is used, resource addresses can include indexes or keys.

Example:

```text
aws_instance.example[0]
```

or:

```text
aws_instance.example["dev"]
```

Understanding resource addresses is important for:

* Reading Terraform plan output
* Understanding state
* Using Terraform CLI commands
* Troubleshooting resources

---

## 21. File Organization

Terraform automatically loads configuration files with the `.tf` extension in the current working directory.

For example:

```text
terraform/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
└── locals.tf
```

Terraform treats these files as part of the same module.

The configuration is conceptually combined:

```text
             Terraform Module
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    main.tf    variables.tf   outputs.tf
       │            │            │
       └────────────┼────────────┘
                    ▼
                Terraform
```

File names help organize configuration for humans.

Terraform does not require resources to be placed in a specific file.

For example, a resource can technically be placed in `variables.tf`, but doing so would reduce clarity.

Therefore, file organization should follow logical responsibilities.

---

## 22. Terraform Naming Conventions

Consistent naming improves readability.

A common approach is to use descriptive local names.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

Instead of:

```hcl
resource "aws_dynamodb_table" "table1" {
  ...
}
```

Good names should communicate purpose.

Examples:

```text
employee
api
lambda
dynamodb
execution_role
```

Avoid unnecessary names such as:

```text
resource1
test123
thing
abc
```

Naming conventions become increasingly important when Terraform projects grow into multiple modules and environments.

---

## 23. Formatting Terraform Configuration

Terraform provides a built-in formatter.

Run:

```bash
terraform fmt
```

For a complete project:

```bash
terraform fmt -recursive
```

Formatting provides consistent indentation and layout.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

Consistent formatting improves:

* Readability
* Code reviews
* Collaboration
* Maintainability

Formatting should be part of the normal Terraform workflow.

---

## 24. Validation of Configuration

Terraform provides configuration validation through:

```bash
terraform validate
```

This checks whether the configuration is syntactically valid and internally consistent.

A typical workflow is:

```bash
terraform fmt -recursive
terraform validate
terraform plan
```

The responsibilities are different:

```text
terraform fmt
      │
      └── Formatting

terraform validate
      │
      └── Configuration validation

terraform plan
      │
      └── Proposed infrastructure changes
```

All three are useful during development.

---

## 25. Configuration Language in the Employee Management API

The Employee Management API project uses HCL to define its AWS infrastructure.

For example, a DynamoDB resource can be defined using:

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

Here multiple Terraform language concepts are used together:

```text
resource block
      │
      ├── arguments
      │
      ├── variable reference
      │
      └── nested block
```

This demonstrates how Terraform configuration language constructs combine to describe infrastructure.

---

## 26. Configuration Language and Modules

Modules use the same Terraform Configuration Language as the root configuration.

For example:

```hcl
module "dynamodb" {
  source = "../modules/dynamodb"

  table_name = var.table_name
}
```

The module call contains:

* A module block
* A source argument
* An input value
* A variable reference

Inside the module, resources are defined using normal Terraform configuration.

```text
Root Configuration
       │
       ▼
Module Block
       │
       ▼
Child Module
       │
       ▼
Resources
```

This allows the same configuration language to be used at different levels of infrastructure design.

---

## 27. Configuration Language and Dependencies

Terraform references can create implicit dependencies.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}

resource "example_resource" "consumer" {
  table_arn = aws_dynamodb_table.employee.arn
}
```

Because the second resource references:

```text
aws_dynamodb_table.employee.arn
```

Terraform understands that the consumer depends on the DynamoDB resource.

Conceptually:

```text
DynamoDB
   │
   │ table_arn
   ▼
Consumer Resource
```

This is an important feature of Terraform's configuration language.

Detailed dependency concepts are documented separately in:

`Terraform/dependencies-and-lifecycle.md`

---

## 28. Configuration vs Imperative Scripts

Terraform configuration should describe infrastructure rather than reproduce a sequence of manual commands.

For example, instead of writing a script that says:

```text
1. Create DynamoDB
2. Wait
3. Create IAM role
4. Attach policy
5. Create Lambda
```

Terraform configuration describes the required infrastructure:

```text
DynamoDB
IAM Role
IAM Policy
Lambda
```

Terraform then determines the appropriate dependency order.

This declarative approach makes the configuration easier to reason about and maintain.

---

## 29. Best Practices

### Keep Configuration Readable

Prefer simple expressions over unnecessarily complicated logic.

### Use Variables for Configurable Values

Avoid hardcoding values that need to change between environments.

### Use Descriptive Names

Resource and variable names should communicate their purpose.

### Use Locals Carefully

Use locals when they improve reuse or readability.

### Avoid Unnecessary Dynamic Blocks

Do not use dynamic blocks when normal blocks are clearer.

### Prefer References Over Hardcoded Relationships

Use resource and module references to allow Terraform to understand dependencies.

### Format Consistently

Run:

```bash
terraform fmt -recursive
```

### Validate Before Planning

Run:

```bash
terraform validate
```

### Keep Modules Understandable

Terraform configuration should remain readable even when reusable modules are introduced.

---

## 30. Learned vs Implemented

### 📚 Learned

The following Terraform Configuration Language concepts were learned:

* HCL
* Terraform configuration structure
* Blocks
* Arguments
* Expressions
* References
* Attribute references
* Strings
* Numbers
* Booleans
* Collections
* Comments
* Variables
* Local values
* Functions
* Conditional expressions
* For expressions
* Dynamic blocks
* Meta-arguments
* Resource addressing
* Terraform file organization
* Naming conventions
* Formatting
* Configuration validation

### 🛠️ Implemented

The Employee Management API Terraform configuration uses HCL to define infrastructure.

The project uses Terraform language constructs including:

* Resource blocks
* Provider blocks
* Variable blocks
* Output blocks
* Module blocks
* Resource references
* Variable references
* Nested blocks
* Terraform expressions
* Terraform formatting and validation

The configuration language is used as the foundation for reusable Terraform infrastructure.

### 🔮 Future Improvements

More advanced Terraform configuration techniques can be introduced as the project grows, including:

* Advanced collection transformations
* Complex object types
* Advanced `for_each` patterns
* Advanced dynamic blocks
* Complex module interfaces
* Advanced lifecycle configuration
* Policy as Code
* Terraform testing
* Security-focused Terraform validation

---

## 31. Key Takeaways

Terraform Configuration Language provides the structure used to describe infrastructure as code.

The most important building blocks are:

```text
HCL
 │
 ├── Blocks
 │     ├── Resources
 │     ├── Providers
 │     ├── Variables
 │     ├── Outputs
 │     ├── Modules
 │     └── Data Sources
 │
 ├── Arguments
 │
 ├── Expressions
 │
 ├── References
 │
 ├── Functions
 │
 ├── Collections
 │
 └── Meta-Arguments
```

The key idea is:

```text
Configuration
      │
      ▼
Desired Infrastructure
      │
      ▼
Terraform
      │
      ▼
Plan
      │
      ▼
Apply
```

Understanding HCL and Terraform Configuration Language is essential because every Terraform project is built using these constructs.

The Employee Management API project applies these concepts progressively, beginning with simple resources and variables and later introducing modules, dependencies, and environment-specific configuration.

This provides the foundation required for writing clean, reusable, and maintainable Terraform infrastructure.
