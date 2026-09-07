# Sprint 8 — Terraform Module Inputs and Outputs

## Overview

Sprint 8 focused on Terraform module inputs and outputs.

Modules become truly reusable when they can receive configuration from the caller and expose useful information back to the caller.

Terraform variables provide inputs to modules, while Terraform outputs expose values from modules.

This creates a clear interface between the root configuration and child modules.

---

## 1. Module Communication

Terraform modules communicate using inputs and outputs.

The basic flow is:

```text
Input Variables
       ↓
     Module
       ↓
     Outputs
```

Variables allow the caller to configure the module.

Outputs allow the module to expose useful information.

---

## 2. Module Input Variables

A module can define variables that control its configuration.

For example:

```hcl
variable "table_name" {

  description = "Name of the DynamoDB table"

  type = string
}
```

Another variable can define the partition key:

```hcl
variable "hash_key" {

  description = "DynamoDB partition key"

  type = string
}
```

The module can then use these values:

```hcl
resource "aws_dynamodb_table" "this" {

  name     = var.table_name
  hash_key = var.hash_key
}
```

---

## 3. Variable Types

Terraform variables should use appropriate types.

For example:

```hcl
variable "function_name" {

  type = string
}
```

For a list:

```hcl
variable "subnet_ids" {

  type = list(string)
}
```

For a map:

```hcl
variable "environment_variables" {

  type = map(string)
}
```

Using types helps Terraform identify invalid configuration.

---

## 4. Variable Descriptions

Variables should have clear descriptions.

For example:

```hcl
variable "runtime" {

  description = "Runtime used by the Lambda function"

  type = string
}
```

Descriptions make modules easier for other developers to understand and reuse.

---

## 5. Variable Validation

Terraform variables can also contain validation rules.

For example:

```hcl
variable "runtime" {

  description = "Lambda runtime"

  type = string

  validation {

    condition = can(
      regex("^python[0-9]+\\.[0-9]+$", var.runtime)
    )

    error_message = "Runtime must be a valid Python Lambda runtime."
  }
}
```

Validation helps prevent invalid values from reaching the infrastructure.

---

## 6. Module Outputs

Outputs expose useful values from a module.

For example:

```hcl
output "table_name" {

  description = "Name of the DynamoDB table"

  value = aws_dynamodb_table.this.name
}
```

Another output can expose the table ARN:

```hcl
output "table_arn" {

  description = "ARN of the DynamoDB table"

  value = aws_dynamodb_table.this.arn
}
```

These values can then be consumed by the root module or other modules.

---

## 7. Passing Outputs Between Modules

One module can use the output of another module.

For example:

```text
DynamoDB Module
       │
       │ table ARN
       ↓
Root Module
       │
       │ table ARN
       ↓
IAM Module
```

The DynamoDB module can expose:

```hcl
output "table_arn" {

  value = aws_dynamodb_table.this.arn
}
```

The root configuration can then use:

```hcl
module.dynamodb.table_arn
```

For example:

```hcl
module "iam" {

  source = "../../modules/iam"

  dynamodb_table_arn = module.dynamodb.table_arn
}
```

---

## 8. Module Dependency Through Outputs

Module outputs can also create Terraform dependencies.

For example:

```text
DynamoDB Module
       │
       │ table_arn
       ↓
IAM Module
       │
       │ role_arn
       ↓
Lambda Module
```

The references create relationships that Terraform can understand automatically.

This combines the concepts learned in Sprint 6 with the module architecture introduced in Sprint 7.

---

## 9. Removing Hardcoded Values

Hardcoding values inside reusable modules reduces flexibility.

Instead of:

```hcl
function_name = "employee-api-dev"
```

inside the module, the module can use:

```hcl
function_name = var.function_name
```

The caller provides:

```hcl
module "lambda" {

  source = "../../modules/lambda"

  function_name = "employee-api-dev"
}
```

This allows the same module to be reused with different values.

---

## 10. Module Interfaces

A reusable module should have a clear interface.

For example, the Lambda module may expose:

```text
Inputs

function_name
runtime
handler
filename
role_arn
environment_variables
```

and:

```text
Outputs

function_name
function_arn
invoke_arn
```

The internal implementation remains inside the module.

The caller only needs to know which inputs are required and which outputs are available.

---

## 11. Application to Employee Management API

The Employee Management API Terraform project uses module variables and outputs to connect infrastructure components.

A simplified flow is:

```text
Environment Configuration
          │
          ↓
      Module Inputs
          │
          ↓
      Terraform Module
          │
          ↓
      Module Outputs
          │
          ↓
   Other Modules / Root
```

For example:

```text
DynamoDB
   ↓
table ARN
   ↓
IAM
   ↓
role ARN
   ↓
Lambda
```

This allows the infrastructure to communicate through Terraform references instead of hardcoded resource identifiers.

---

## 12. Learned vs Implemented

### 📚 Learned

* Module input variables
* Variable types
* Variable descriptions
* Variable validation
* Module outputs
* Resource attributes
* Module-to-module communication
* Module interfaces
* Removing hardcoded values

### 🛠️ Implemented

The Employee Management API modules were structured to accept configurable values through variables.

Useful resource information was exposed through module outputs.

Outputs such as resource ARNs and names can be consumed by other parts of the Terraform configuration.

Module references also create implicit dependencies between infrastructure components.

### 🔮 Future Improvements

Module interfaces can be improved by:

* Adding stronger variable validation.
* Documenting every module input and output.
* Exposing only required outputs.
* Marking appropriate sensitive outputs as sensitive.
* Reducing unnecessary module inputs.
* Standardizing module interfaces across the project.

---

## 13. Key Takeaways

> Module variables provide configuration to reusable modules.

> Module outputs expose useful information from modules.

> Variables and outputs create a clean interface between Terraform components.

> Module outputs can also create implicit dependencies.

> Removing hardcoded values makes Terraform modules more reusable.

Sprint 8 established clean module interfaces and prepared the project for **environment-aware and reusable infrastructure** in Sprint 9.
