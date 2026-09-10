# Terraform Outputs

## Overview

Terraform outputs provide a way to expose useful values from a Terraform configuration.

Outputs are commonly used to:

* Display important infrastructure values after deployment
* Expose values from child modules
* Pass information between modules
* Provide information for users and automation
* Retrieve resource attributes
* Integrate Terraform with CI/CD pipelines
* Make infrastructure information available without manually inspecting resources

For example, after creating an AWS DynamoDB table, Terraform can expose its ARN:

```hcl
output "table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.employee.arn
}
```

After `terraform apply`, Terraform can display the value.

The key idea is:

> **Variables provide inputs to Terraform, while outputs expose useful values from Terraform.**

---

## 1. What Is a Terraform Output?

A Terraform output is a named value that Terraform exposes from a module.

Outputs can expose:

* Resource attributes
* Computed values
* Module outputs
* Derived expressions
* IDs
* ARNs
* URLs
* Names
* Endpoints
* Other useful infrastructure information

Example:

```hcl
output "employee_table_name" {
  description = "Name of the employee DynamoDB table"
  value       = aws_dynamodb_table.employee.name
}
```

Terraform evaluates the expression in `value` and makes the result available as an output.

---

## 2. Why Use Outputs?

Outputs are useful because Terraform creates infrastructure containing values that may not be known until Terraform evaluates or creates the infrastructure.

For example:

```text
Terraform Configuration
        │
        ▼
AWS Resources
        │
        ├── Resource ID
        ├── ARN
        ├── Name
        └── Endpoint
                │
                ▼
          Terraform Output
```

Instead of manually finding these values in the AWS console, Terraform can expose them.

Outputs are especially useful for:

* Module composition
* Infrastructure discovery
* Deployment information
* Automation
* CI/CD
* Troubleshooting
* Integration with other systems

---

## 3. Output Block Syntax

The basic syntax is:

```hcl
output "name" {
  value = expression
}
```

Example:

```hcl
output "table_name" {
  value = aws_dynamodb_table.employee.name
}
```

A more descriptive output:

```hcl
output "table_name" {
  description = "Name of the employee DynamoDB table"
  value       = aws_dynamodb_table.employee.name
}
```

---

## 4. Output Name

The output name identifies the output.

Example:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

Here:

```text
output
  │
  └── table_arn
```

The output can later be retrieved using:

```bash
terraform output table_arn
```

Output names should be:

* Descriptive
* Consistent
* Meaningful
* Stable

Prefer:

```hcl
output "employee_table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

Instead of:

```hcl
output "value1" {
  value = aws_dynamodb_table.employee.arn
}
```

---

## 5. The `value` Argument

The `value` argument defines what the output exposes.

Example:

```hcl
output "table_name" {
  value = aws_dynamodb_table.employee.name
}
```

The value can come from:

* Resources
* Data sources
* Variables
* Locals
* Module outputs
* Expressions
* Functions

Example using a variable:

```hcl
variable "environment" {
  type = string
}

output "environment" {
  value = var.environment
}
```

---

## 6. Resource Attributes as Outputs

A common use case is exposing resource attributes.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"

  hash_key = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}

output "table_name" {
  value = aws_dynamodb_table.employee.name
}

output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

Possible outputs include:

```text
table_name
table_arn
```

---

## 7. Output Descriptions

The `description` argument explains what the output represents.

Example:

```hcl
output "table_arn" {
  description = "ARN of the employee DynamoDB table"
  value       = aws_dynamodb_table.employee.arn
}
```

Descriptions improve readability and documentation.

For reusable modules, descriptions are especially useful because consumers may not know what every output represents.

---

## 8. Sensitive Outputs

Some outputs may contain sensitive information.

Terraform supports:

```hcl
output "database_password" {
  description = "Database password"
  value       = var.database_password
  sensitive   = true
}
```

The `sensitive` argument tells Terraform to avoid displaying the value normally in CLI output.

Example:

```hcl
output "secret_value" {
  value     = var.secret_value
  sensitive = true
}
```

Terraform will indicate that the value is sensitive rather than displaying it normally.

### Important

`sensitive = true` does **not** encrypt or remove the value from Terraform state.

Therefore:

> **Sensitive outputs still require proper state security.**

Detailed state and secret-management practices are covered separately in the Terraform state and security documentation.

---

## 9. Output References

Outputs can reference resources using Terraform references.

Example:

```hcl
output "lambda_function_name" {
  value = aws_lambda_function.employee_api.function_name
}
```

The reference:

```text
aws_lambda_function.employee_api.function_name
```

means:

```text
resource type
    │
    ▼
aws_lambda_function
    │
    ▼
resource name
    │
    ▼
employee_api
    │
    ▼
attribute
    │
    ▼
function_name
```

---

## 10. Outputs from Data Sources

Outputs can also expose values returned by data sources.

Example:

```hcl
data "aws_region" "current" {}

output "aws_region" {
  value = data.aws_region.current.name
}
```

Another example:

```hcl
data "aws_caller_identity" "current" {}

output "aws_account_id" {
  value = data.aws_caller_identity.current.account_id
}
```

This can be useful for displaying information about the environment in which Terraform is running.

---

## 11. Outputs Using Variables

Variables can be exposed through outputs.

Example:

```hcl
variable "environment" {
  type = string
}

output "environment" {
  description = "Current deployment environment"
  value       = var.environment
}
```

If the environment is:

```text
dev
```

Terraform can expose:

```text
environment = "dev"
```

However, outputs should expose values that are actually useful.

Do not create outputs for every variable without a reason.

---

## 12. Outputs Using Locals

Outputs can reference locals.

Example:

```hcl
locals {
  table_name = "employee-${var.environment}-table"
}

output "table_name" {
  value = local.table_name
}
```

The flow is:

```text
Variable
   │
   ▼
Local Value
   │
   ▼
Output
```

---

## 13. Root Module Outputs

The root module is the Terraform configuration where commands such as:

```bash
terraform plan
terraform apply
```

are normally executed.

Root modules can define outputs.

Example:

```hcl
output "employee_table_name" {
  description = "Employee DynamoDB table name"
  value       = aws_dynamodb_table.employee.name
}
```

After applying the configuration:

```bash
terraform output
```

can display the root module outputs.

---

## 14. Child Module Outputs

Child modules can also define outputs.

Example module:

```text
modules/
└── dynamodb/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

The module's `outputs.tf` may contain:

```hcl
output "table_name" {
  description = "Name of the DynamoDB table"
  value       = aws_dynamodb_table.this.name
}

output "table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.this.arn
}
```

These outputs become part of the module's interface.

---

## 15. Calling Child Module Outputs

Suppose the root module calls the DynamoDB module:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
}
```

The root module can reference the child module's output:

```hcl
module.dynamodb.table_arn
```

Example:

```hcl
output "employee_table_arn" {
  description = "ARN of the employee DynamoDB table"
  value       = module.dynamodb.table_arn
}
```

The data flow is:

```text
DynamoDB Resource
       │
       ▼
DynamoDB Module Output
       │
       ▼
module.dynamodb.table_arn
       │
       ▼
Root Module Output
```

---

## 16. Outputs Between Modules

Outputs are one of the main mechanisms for passing information between modules.

Example:

```text
DynamoDB Module
      │
      │ table_arn
      ▼
IAM Module
      │
      │ policy references table ARN
      ▼
Lambda Module
```

The root module coordinates these connections.

Example:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"
}

module "iam" {
  source = "../../modules/iam"

  table_arn = module.dynamodb.table_arn
}
```

This creates a dependency between the modules.

---

## 17. Outputs and Dependencies

Output references can participate in Terraform's dependency graph.

Example:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

If another module uses:

```hcl
module.dynamodb.table_arn
```

Terraform understands the relationship.

Conceptually:

```text
DynamoDB
   │
   ▼
DynamoDB Output
   │
   ▼
IAM Policy
```

Terraform can therefore determine the required ordering from references.

Explicit `depends_on` should generally not be added when a normal reference already establishes the dependency.

---

## 18. Outputs and `terraform plan`

Terraform can calculate output values during planning when the required information is already known.

Some values may remain unknown until apply.

Example:

```text
terraform plan

Outputs:
  table_name = "employee-dev-table"
  table_arn  = (known after apply)
```

An output can therefore contain:

```text
known value
```

or:

```text
known after apply
```

depending on when Terraform can determine the value.

---

## 19. Outputs and `terraform apply`

After:

```bash
terraform apply
```

Terraform displays output values.

Example:

```text
Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Outputs:

employee_table_name = "employee-dev-table"
employee_table_arn  = "arn:aws:dynamodb:..."
```

This makes outputs useful as a deployment summary.

---

## 20. Viewing Outputs

To view all outputs:

```bash
terraform output
```

Example:

```text
employee_table_name = "employee-dev-table"
employee_table_arn = "arn:aws:dynamodb:..."
```

To retrieve one output:

```bash
terraform output employee_table_name
```

---

## 21. Using `terraform output -raw`

For simple string outputs, `-raw` can return the raw value without Terraform's display formatting.

Example:

```bash
terraform output -raw employee_table_name
```

Instead of a Terraform-formatted representation, the command returns:

```text
employee-dev-table
```

This is useful in shell scripts and automation.

`-raw` is intended for values that can be represented as a simple string.

---

## 22. JSON Output

Terraform can return outputs as JSON:

```bash
terraform output -json
```

Example structure:

```json
{
  "employee_table_name": {
    "sensitive": false,
    "type": "string",
    "value": "employee-dev-table"
  }
}
```

JSON output is useful for:

* Scripts
* CI/CD pipelines
* Automation
* Programmatic processing

---

## 23. Outputs and Terraform State

Terraform stores output values as part of Terraform state.

This is important because state can contain infrastructure information and potentially sensitive values.

For example:

```text
Terraform State
     │
     ├── Resources
     ├── Resource attributes
     ├── Module information
     └── Output values
```

Therefore, state should be protected using appropriate access controls and backend security.

The `sensitive` flag changes display behavior, but does not make the underlying state value disappear.

---

## 24. Outputs and State Security

Because outputs can be stored in state, avoid exposing secrets unnecessarily.

Poor practice:

```hcl
output "api_secret" {
  value = var.api_secret
}
```

Better:

```hcl
output "api_endpoint" {
  value = aws_api_gateway_stage.api.invoke_url
}
```

Only expose information that consumers actually need.

For secrets, prefer dedicated secret-management systems rather than using Terraform outputs as a secret distribution mechanism.

---

## 25. Outputs in CI/CD

CI/CD pipelines can consume Terraform outputs.

Example:

```bash
terraform apply -auto-approve
terraform output -json
```

The pipeline can then process the JSON output.

For a deployment pipeline:

```text
Terraform Apply
      │
      ▼
Terraform Outputs
      │
      ▼
CI/CD Pipeline
      │
      ├── Deployment information
      ├── Endpoint
      ├── Resource identifiers
      └── Integration steps
```

This allows later pipeline stages to consume infrastructure information automatically.

---

## 26. Outputs in the Employee Management API

The Employee Management API project contains resources such as:

```text
DynamoDB
IAM
Lambda
API Gateway
CloudWatch
KMS
```

Outputs can expose useful information from these resources.

For example:

```hcl
output "employee_table_name" {
  description = "Name of the employee DynamoDB table"
  value       = module.dynamodb.table_name
}
```

Another example:

```hcl
output "employee_table_arn" {
  description = "ARN of the employee DynamoDB table"
  value       = module.dynamodb.table_arn
}
```

If the Lambda module exposes its function name:

```hcl
output "lambda_function_name" {
  description = "Name of the Employee Management API Lambda function"
  value       = module.lambda.function_name
}
```

If API Gateway is implemented and its module exposes an endpoint:

```hcl
output "api_endpoint" {
  description = "Employee Management API endpoint"
  value       = module.api_gateway.invoke_url
}
```

The exact output depends on which resources and module outputs are currently implemented in the project.

---

## 27. Project Output Flow

The Employee Management API can conceptually use the following flow:

```text
                 Root Module
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
   DynamoDB         IAM          Lambda
       │             │             │
       │             │             │
       ▼             │             ▼
 table_name          │       function_name
 table_arn           │
       │             │
       └──────┬──────┘
              │
              ▼
        Root Outputs
              │
              ▼
       terraform output
```

This keeps infrastructure information discoverable without hardcoding values elsewhere.

---

## 28. Outputs vs Variables

Variables and outputs have opposite roles.

| Feature      | Variables                 | Outputs                     |
| ------------ | ------------------------- | --------------------------- |
| Purpose      | Provide inputs            | Expose values               |
| Direction    | Into configuration        | Out of configuration        |
| Reference    | `var.name`                | `module.name.output` or CLI |
| Common use   | Environment/configuration | IDs, ARNs, URLs             |
| Defined with | `variable`                | `output`                    |

Example:

```hcl
variable "environment" {
  type = string
}
```

Input:

```text
environment → Terraform
```

Output:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

Output:

```text
Terraform → table ARN
```

---

## 29. Outputs vs Locals

Locals are internal reusable expressions.

Example:

```hcl
locals {
  table_name = "employee-${var.environment}-table"
}
```

Outputs expose values outside the module.

```hcl
output "table_name" {
  value = local.table_name
}
```

The distinction is:

```text
Variable
   │
   ▼
Local
   │
   ▼
Resource / Expression
   │
   ▼
Output
```

Locals are primarily internal.

Outputs form part of the module's externally visible interface.

---

## 30. Outputs vs Hardcoded Values

Instead of manually documenting resource identifiers:

```text
DynamoDB table ARN:
arn:aws:dynamodb:...
```

Terraform can expose the value dynamically:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

This is better because Terraform generates the value from the actual infrastructure.

---

## 31. Naming Output Values

Good output names should describe the information being exposed.

Examples:

```hcl
output "table_name" {}

output "table_arn" {}

output "lambda_function_name" {}

output "api_endpoint" {}

output "aws_region" {}
```

Avoid vague names:

```hcl
output "value" {}

output "result" {}

output "output1" {}
```

Good naming makes CLI usage and module composition easier.

---

## 32. Output Design for Reusable Modules

A reusable module should expose only useful information.

For example, a DynamoDB module may expose:

```hcl
output "table_name" {
  value = aws_dynamodb_table.this.name
}

output "table_arn" {
  value = aws_dynamodb_table.this.arn
}

output "table_id" {
  value = aws_dynamodb_table.this.id
}
```

Consumers can then choose what they need.

Avoid exposing every possible resource attribute.

A good module interface should be:

```text
Simple
     +
Useful
     +
Stable
     +
Minimal
```

---

## 33. Common Output Mistakes

### Mistake 1 — Exposing unnecessary values

Do not create outputs for every resource attribute.

Expose values that consumers actually need.

---

### Mistake 2 — Treating sensitive as encryption

This is incorrect:

```hcl
sensitive = true
```

does not mean:

```text
encrypted
```

It primarily controls how Terraform displays the value.

---

### Mistake 3 — Hardcoding values

Avoid:

```hcl
output "table_name" {
  value = "employee-dev-table"
}
```

when Terraform already manages the resource.

Prefer:

```hcl
output "table_name" {
  value = aws_dynamodb_table.employee.name
}
```

---

### Mistake 4 — Using the wrong module output reference

Incorrect:

```hcl
value = module.dynamodb.employee_table_arn
```

if the module actually defines:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.this.arn
}
```

The correct reference is:

```hcl
value = module.dynamodb.table_arn
```

---

### Mistake 5 — Using outputs as secret storage

Terraform outputs should not become a general-purpose secret distribution mechanism.

Use proper secret-management solutions for sensitive credentials.

---

### Mistake 6 — Assuming every output is known during plan

Some values are computed only after resources are created.

Terraform may show:

```text
(known after apply)
```

during planning.

---

## 34. Output Best Practices

Follow these practices:

1. Give outputs descriptive names.
2. Add useful descriptions.
3. Expose only useful values.
4. Prefer resource attributes over hardcoded values.
5. Use outputs to define clean module interfaces.
6. Mark sensitive outputs as `sensitive = true`.
7. Do not treat `sensitive` as encryption.
8. Protect Terraform state.
9. Use `terraform output -raw` for simple string automation.
10. Use `terraform output -json` for structured automation.
11. Avoid duplicating unnecessary outputs.
12. Keep module outputs stable and meaningful.

---

## 35. Practical Output Workflow

A practical workflow is:

```text
1. Create resource
       │
       ▼
2. Identify useful attributes
       │
       ▼
3. Define output
       │
       ▼
4. Run terraform fmt
       │
       ▼
5. Run terraform validate
       │
       ▼
6. Run terraform plan
       │
       ▼
7. Run terraform apply
       │
       ▼
8. Check terraform output
```

Example:

```bash
terraform fmt -recursive
terraform validate
terraform plan
terraform apply
terraform output
```

---

## 36. Practical Example

Consider the Employee Management API DynamoDB module.

### Resource

```hcl
resource "aws_dynamodb_table" "this" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"

  hash_key = "employee_id"

  attribute {
    name = "employee_id"
    type = "S"
  }
}
```

### Module outputs

```hcl
output "table_name" {
  description = "Name of the DynamoDB table"
  value       = aws_dynamodb_table.this.name
}

output "table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.this.arn
}
```

### Root module

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
}
```

### Root output

```hcl
output "employee_table_name" {
  description = "Name of the employee DynamoDB table"
  value       = module.dynamodb.table_name
}

output "employee_table_arn" {
  description = "ARN of the employee DynamoDB table"
  value       = module.dynamodb.table_arn
}
```

The resulting flow is:

```text
var.table_name
      │
      ▼
DynamoDB Module
      │
      ▼
DynamoDB Resource
      │
      ├── name
      └── arn
          │
          ▼
     Module Outputs
          │
          ▼
      Root Outputs
          │
          ▼
 terraform output
```

---

## 37. Outputs and Environment Separation

The project uses environment-aware naming.

For example:

```hcl
variable "environment" {
  type = string
}
```

A module may create:

```hcl
name = "employee-${var.environment}-table"
```

The output automatically reflects the selected environment.

For:

```text
environment = "dev"
```

the output can be:

```text
employee-dev-table
```

For another environment:

```text
employee-test-table
```

The output configuration remains the same while the input changes.

This demonstrates how variables and outputs work together:

```text
Variable
   │
   ▼
Environment-specific configuration
   │
   ▼
Resource
   │
   ▼
Output
```

---

## 38. Outputs and Module Interfaces

A Terraform module can be viewed as having an interface:

```text
             Module
        ┌───────────────┐
        │               │
Inputs ─► Configuration │
        │               │
        │               │
Outputs ◄───────────────┤
        │               │
        └───────────────┘
```

Variables define the input side.

Outputs define the output side.

This makes modules easier to reuse and compose.

---

## 39. Outputs and Infrastructure Composition

Modules can expose values that other modules consume.

Example:

```text
DynamoDB Module
      │
      │ table_arn
      ▼
IAM Module
      │
      │ role_arn
      ▼
Lambda Module
```

This allows Terraform configurations to be composed from smaller reusable modules.

The root module acts as the coordinator.

---

## 40. When to Use Outputs

Use outputs when:

* A resource identifier is useful after deployment.
* A child module needs to expose information.
* Another module needs a resource attribute.
* CI/CD needs deployment information.
* Users need important infrastructure values.
* Automation needs structured Terraform results.

Examples:

```text
ARN
ID
Name
URL
Endpoint
Region
Account ID
```

---

## 41. When Not to Use Outputs

Do not create outputs simply because Terraform allows it.

Avoid outputs when:

* The value is never consumed.
* The value is unnecessary internal information.
* It exposes sensitive information without a valid requirement.
* The same value is already available through a better interface.
* It creates unnecessary module coupling.

The goal is not:

```text
Maximum outputs
```

The goal is:

```text
Useful outputs
```

---

## 42. Current Project Implementation

The Employee Management API project uses reusable Terraform modules and environment-specific configuration.

Outputs fit into this architecture by exposing useful information from resources and child modules.

For example:

```text
Environment Variables
        │
        ▼
Root Module
        │
        ├── DynamoDB Module
        │       │
        │       └── table_name / table_arn
        │
        ├── IAM Module
        │       │
        │       └── role information
        │
        └── Lambda Module
                │
                └── function information
                        │
                        ▼
                  Root Outputs
```

The exact outputs implemented should reflect the resources and module interfaces that currently exist in the project rather than creating outputs only for documentation purposes.

---

## 43. Outputs and the Terraform CLI

Important commands include:

### Show all outputs

```bash
terraform output
```

### Show one output

```bash
terraform output table_name
```

### Show a raw string

```bash
terraform output -raw table_name
```

### Show outputs as JSON

```bash
terraform output -json
```

These commands are useful during development, troubleshooting, and automation.

---

## 44. Validation Checklist

Before committing output documentation or implementation changes, verify:

```bash
terraform fmt -recursive
terraform validate
terraform plan
```

After applying:

```bash
terraform output
```

For automation:

```bash
terraform output -json
```

Check that:

* Output names are meaningful.
* Values reference the correct resources/modules.
* Descriptions are clear.
* Sensitive outputs are marked appropriately.
* No unnecessary secrets are exposed.
* Outputs match the current module interface.

---

## 45. Learned vs Implemented

### 📚 Learned

The following concepts were learned:

* Terraform output blocks
* Output names
* `value`
* `description`
* Sensitive outputs
* Resource attributes as outputs
* Data source outputs
* Variable and local references
* Root module outputs
* Child module outputs
* Module-to-module value passing
* Output dependencies
* `terraform output`
* `terraform output -raw`
* `terraform output -json`
* Outputs and Terraform state
* Outputs in CI/CD
* Output design and best practices

### 🛠️ Implemented

The documentation establishes how outputs fit into the existing Employee Management API Terraform architecture.

The project already uses reusable modules, so outputs can expose useful module values such as:

```text
DynamoDB table name
DynamoDB table ARN
Lambda function name
Other resource identifiers
```

The exact outputs should be implemented only where they provide value to the current project.

### 🔮 Future Improvements

Potential future improvements include:

* Standardizing outputs across all modules
* Adding API Gateway endpoint outputs when implemented
* Adding Lambda ARN outputs
* Adding IAM role ARN outputs where useful
* Consuming outputs in CI/CD
* Improving output conventions across environments
* Integrating Terraform outputs with deployment automation
* Strengthening sensitive-output and state-security practices

---

## 46. Key Takeaways

1. Outputs expose useful values from Terraform configurations.
2. Outputs are declared using the `output` block.
3. The `value` argument determines what is exposed.
4. `description` improves readability.
5. `sensitive = true` hides values from normal CLI display but does not remove them from state.
6. Child modules use outputs to expose values to their callers.
7. Root modules can expose child-module outputs.
8. Output references can participate in Terraform's dependency graph.
9. `terraform output` displays stored output values.
10. `terraform output -raw` is useful for simple string automation.
11. `terraform output -json` is useful for programmatic consumption.
12. Outputs should be meaningful and minimal.
13. Terraform state must be protected because output values can be stored in state.
14. Variables provide inputs; outputs expose results.
15. Good module outputs create clean and reusable module interfaces.

> **Core Principle:** Use Terraform outputs to expose meaningful infrastructure information, create clean module interfaces, and provide reliable values for users and automation without exposing unnecessary or sensitive data.
