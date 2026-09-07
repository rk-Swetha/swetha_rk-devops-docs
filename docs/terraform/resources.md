# Terraform Resources

## 1. Overview

Terraform resources are the primary building blocks used to create and manage infrastructure.

A resource represents a real infrastructure object such as:

* AWS DynamoDB table
* AWS Lambda function
* IAM role
* IAM policy
* API Gateway
* S3 bucket
* VPC
* Security group

Terraform uses resource blocks to describe the desired configuration of these objects.

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

Terraform compares the configuration with the current infrastructure and determines what actions are required to make the real infrastructure match the desired configuration.

---

## 2. What Is a Terraform Resource?

A Terraform resource represents an infrastructure object that Terraform can create, update, or destroy.

The general structure is:

```hcl
resource "<RESOURCE_TYPE>" "<LOCAL_NAME>" {
  argument = value
}
```

Example:

```hcl
resource "aws_s3_bucket" "application" {
  bucket = "my-application-bucket"
}
```

Here:

| Component                 | Meaning                  |
| ------------------------- | ------------------------ |
| `resource`                | Terraform resource block |
| `aws_s3_bucket`           | Resource type            |
| `application`             | Local resource name      |
| `bucket`                  | Resource argument        |
| `"my-application-bucket"` | Desired value            |

The resource type tells Terraform **what kind of infrastructure object** should exist.

The local name identifies the resource within the Terraform configuration.

---

## 3. Resource Types

A resource type determines the kind of infrastructure object Terraform manages.

Resource types are usually provided by Terraform providers.

Examples from the AWS provider include:

```text
aws_lambda_function
aws_dynamodb_table
aws_iam_role
aws_iam_policy
aws_api_gateway_rest_api
aws_cloudwatch_log_group
```

For example:

```hcl
resource "aws_iam_role" "lambda" {
  name = "employee-api-dev-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Principal = {
        Service = "lambda.amazonaws.com"
      }

      Action = "sts:AssumeRole"
    }]
  })
}
```

Terraform understands how to manage `aws_iam_role` because the AWS provider defines that resource type.

---

## 4. Resource Local Names

The second label in a resource block is the local resource name.

Example:

```hcl
resource "aws_lambda_function" "employee_api" {
  ...
}
```

Here:

```text
aws_lambda_function → resource type
employee_api        → local name
```

The local name is used when referring to the resource elsewhere in the configuration.

For example:

```hcl
aws_lambda_function.employee_api.arn
```

The local name does not necessarily become the actual AWS resource name.

For example:

```hcl
resource "aws_lambda_function" "employee_api" {
  function_name = "employee-api-dev"
}
```

Terraform's local name is:

```text
employee_api
```

while the actual AWS Lambda function name is:

```text
employee-api-dev
```

---

## 5. Resource Arguments

Arguments configure the behavior and properties of a resource.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

The arguments are:

```text
name
billing_mode
hash_key
```

Arguments can contain:

* literal values
* variables
* local values
* resource references
* function calls
* conditional expressions

Example using a variable:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = var.billing_mode
  hash_key     = var.hash_key
}
```

This makes the resource configuration reusable across environments.

---

## 6. Resource Attributes

A resource exposes attributes that can be referenced by other Terraform configuration.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

The resource can expose attributes such as:

```text
aws_dynamodb_table.employee.id
aws_dynamodb_table.employee.arn
aws_dynamodb_table.employee.name
```

These attributes can be passed to other resources or outputs.

Example:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

This allows Terraform configuration to connect different infrastructure components.

---

## 7. Resource References

Terraform resources can reference other resources.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

Another resource can reference the table:

```hcl
resource "aws_lambda_function" "employee_api" {
  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.employee.name
    }
  }
}
```

The expression:

```hcl
aws_dynamodb_table.employee.name
```

means:

> Use the `name` attribute of the `employee` DynamoDB resource.

Terraform also understands that the Lambda configuration depends on the DynamoDB resource.

This is an example of an **implicit dependency**.

Detailed dependency graph behavior is covered separately in Issue #22.

---

## 8. Resource Addressing

Terraform identifies resources using resource addresses.

The basic format is:

```text
<RESOURCE_TYPE>.<LOCAL_NAME>
```

Example:

```text
aws_lambda_function.employee_api
```

A specific attribute can be referenced using:

```text
aws_lambda_function.employee_api.arn
```

Resource addresses are also used with Terraform CLI commands.

For example:

```bash
terraform state show aws_lambda_function.employee_api
```

This allows Terraform to target a specific resource.

---

## 9. Resource Creation

When a resource exists in the configuration but does not yet exist in the infrastructure, Terraform can create it.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

Running:

```bash
terraform plan
```

may show:

```text
+ aws_dynamodb_table.employee
```

The `+` symbol indicates that Terraform plans to create the resource.

Running:

```bash
terraform apply
```

allows Terraform to create the resource.

---

## 10. Resource Updates

Terraform can also update resources when supported attributes change.

For example:

```hcl
resource "aws_cloudwatch_log_group" "lambda" {
  name              = "/aws/lambda/employee-api-dev"
  retention_in_days = 30
}
```

If the retention period changes:

```hcl
retention_in_days = 90
```

Terraform compares the configuration with the existing infrastructure and may plan an update.

Example plan notation:

```text
~ aws_cloudwatch_log_group.lambda
```

The `~` indicates that Terraform plans to modify the resource.

Not every attribute can be changed in place.

Some changes require the resource to be replaced.

---

## 11. Resource Replacement

Some resource changes cannot be performed as an in-place update.

Terraform may therefore destroy the existing resource and create a replacement.

Example plan notation:

```text
-/+ aws_example_resource.example
```

This indicates replacement.

The exact behavior depends on the resource type and provider implementation.

A replacement can be important because it may cause:

* temporary downtime
* data loss
* changed resource identifiers
* dependency changes

Therefore, resource replacement should always be reviewed carefully in `terraform plan`.

---

## 12. Resource Destruction

If a resource is removed from the Terraform configuration, Terraform may determine that the resource should no longer be managed.

For example, if this resource is removed:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

Terraform may plan:

```text
- aws_dynamodb_table.employee
```

The `-` indicates that Terraform plans to destroy the resource.

Running:

```bash
terraform apply
```

would apply that planned destruction.

For production infrastructure, destructive changes should always be reviewed carefully before applying them.

---

## 13. Resource Lifecycle

Terraform provides lifecycle settings that can influence how resource changes are handled.

The lifecycle block is configured inside a resource:

```hcl
resource "aws_s3_bucket" "application" {
  bucket = "my-application-bucket"

  lifecycle {
    ...
  }
}
```

Common lifecycle arguments include:

```text
create_before_destroy
prevent_destroy
ignore_changes
```

These controls should be used intentionally rather than added everywhere.

Detailed lifecycle behavior will be explored further in Issue #22.

---

## 14. `create_before_destroy`

The `create_before_destroy` lifecycle rule tells Terraform to create a replacement before destroying the existing resource when replacement is required.

Example:

```hcl
resource "aws_example_resource" "application" {
  ...

  lifecycle {
    create_before_destroy = true
  }
}
```

Conceptually:

Without this setting:

```text
Destroy old
    ↓
Create new
```

With this setting:

```text
Create new
    ↓
Destroy old
```

This can reduce downtime when the resource supports having both versions temporarily.

However, it is not suitable for every resource because some infrastructure objects require unique names.

---

## 15. `prevent_destroy`

`prevent_destroy` prevents Terraform from destroying a resource through normal Terraform operations.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"

  lifecycle {
    prevent_destroy = true
  }
}
```

This is useful for resources where accidental deletion would be especially dangerous.

However, it should not be treated as a replacement for backups or other disaster-recovery mechanisms.

---

## 16. `ignore_changes`

`ignore_changes` tells Terraform to ignore changes to specified resource attributes when evaluating configuration drift.

Example:

```hcl
resource "aws_example_resource" "application" {
  ...

  lifecycle {
    ignore_changes = [
      tags
    ]
  }
}
```

This can be useful when another trusted system is expected to manage a particular attribute.

However, excessive use can hide configuration drift.

Use it only when there is a clear reason.

---

## 17. `count`

`count` allows Terraform to create multiple instances of a resource based on a number.

Example:

```hcl
resource "aws_s3_bucket" "application" {
  count = 2

  bucket = "employee-api-${count.index}"
}
```

Terraform creates:

```text
aws_s3_bucket.application[0]
aws_s3_bucket.application[1]
```

`count.index` provides the index of the current instance.

`count` is useful when instances are nearly identical and can naturally be represented by numeric indexes.

---

## 18. `for_each`

`for_each` creates resource instances from a collection.

Example:

```hcl
resource "aws_s3_bucket" "application" {
  for_each = toset([
    "logs",
    "backups"
  ])

  bucket = "employee-api-${each.key}"
}
```

Terraform creates instances such as:

```text
aws_s3_bucket.application["logs"]
aws_s3_bucket.application["backups"]
```

`for_each` is often preferable when each instance has a meaningful identifier.

Example using a map:

```hcl
variable "buckets" {
  type = map(string)

  default = {
    logs    = "employee-api-logs"
    backups = "employee-api-backups"
  }
}
```

Then:

```hcl
resource "aws_s3_bucket" "application" {
  for_each = var.buckets

  bucket = each.value
}
```

---

## 19. `count` vs `for_each`

Both `count` and `for_each` support multiple resource instances.

### `count`

Best suited for:

```text
Similar resources
Numeric instances
Simple conditional creation
```

Example:

```hcl
count = var.create_bucket ? 1 : 0
```

### `for_each`

Best suited for:

```text
Meaningful identifiers
Maps
Sets
Different configuration per instance
```

Example:

```hcl
for_each = var.environments
```

A common rule is:

> Use `count` when instances are naturally index-based; use `for_each` when instances have meaningful keys.

---

## 20. Resource Dependencies

Terraform automatically determines dependencies when one resource references another.

Example:

```hcl
resource "aws_iam_role" "lambda" {
  ...
}

resource "aws_lambda_function" "employee_api" {
  role = aws_iam_role.lambda.arn
}
```

Terraform sees:

```text
IAM Role
   ↓
Lambda Function
```

The Lambda function depends on the IAM role.

This is an **implicit dependency**.

Terraform can also support explicit dependencies using:

```hcl
depends_on = [
  aws_example_resource.example
]
```

Explicit dependencies should only be used when Terraform cannot infer the dependency from configuration.

Detailed dependency graph concepts belong to Issue #22.

---

## 21. Resource Providers

A resource type belongs to a provider.

For example:

```hcl
resource "aws_lambda_function" "employee_api" {
  ...
}
```

belongs to the AWS provider.

The provider supplies Terraform with the implementation required to manage the AWS resource.

Provider configuration may look like:

```hcl
provider "aws" {
  region = var.aws_region
}
```

Terraform then uses the AWS provider to communicate with AWS.

The relationship is:

```text
Terraform Configuration
        ↓
      Provider
        ↓
      Resource
        ↓
Cloud Infrastructure
```

Provider concepts are documented separately in `Terraform/providers.md`.

---

## 22. Resources in Modules

Resources are commonly defined inside reusable Terraform modules.

Example:

```text
terraform/
├── modules/
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
```

The DynamoDB module may contain:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = var.billing_mode
  hash_key     = var.hash_key
}
```

The root module can then call it:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name   = var.table_name
  billing_mode = var.billing_mode
  hash_key     = var.hash_key
}
```

This separates reusable resource definitions from environment-specific configuration.

---

## 23. Resources in the Employee Management API

The Employee Management API uses Terraform to manage AWS infrastructure.

The core infrastructure includes resources such as:

```text
API Gateway
     │
     ▼
Lambda Function
     │
     ▼
DynamoDB Table
```

IAM resources provide the required permissions:

```text
Lambda
   │
   ▼
IAM Role
   │
   └── IAM Policies
```

A simplified Terraform structure is:

```text
terraform/
│
├── modules/
│   ├── dynamodb/
│   │   └── resources
│   │
│   ├── iam/
│   │   └── resources
│   │
│   └── lambda/
│       └── resources
│
└── environments/
    └── dev/
        └── configuration
```

The project uses Terraform resources to represent AWS infrastructure instead of creating those resources manually through the AWS Console.

---

## 24. Example: DynamoDB Resource

The project uses a DynamoDB resource similar to:

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

The resource definition describes the desired DynamoDB configuration.

The actual table is created by Terraform when the configuration is applied.

---

## 25. Example: IAM Role Resource

A Lambda function requires an IAM execution role.

Example:

```hcl
resource "aws_iam_role" "lambda" {
  name = var.role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Principal = {
        Service = "lambda.amazonaws.com"
      }

      Action = "sts:AssumeRole"
    }]
  })
}
```

The IAM role becomes a managed Terraform resource.

The Lambda resource can then reference the role:

```hcl
role = aws_iam_role.lambda.arn
```

This creates a relationship between the two resources.

---

## 26. Example: Lambda Resource

A Lambda function can be represented as:

```hcl
resource "aws_lambda_function" "employee_api" {
  function_name = var.function_name
  runtime       = var.runtime
  handler       = var.handler

  filename = var.filename

  role = aws_iam_role.lambda.arn
}
```

Important resource arguments include:

```text
function_name
runtime
handler
filename
role
```

The role reference:

```hcl
aws_iam_role.lambda.arn
```

connects the Lambda resource to the IAM role.

---

## 27. Resources and Terraform Plan

`terraform plan` is one of the most important commands for understanding resource behavior.

Example:

```bash
terraform plan
```

Terraform may show:

```text
+ create
~ update
- destroy
-/+ replace
```

Meaning:

| Symbol | Meaning |
| ------ | ------- |
| `+`    | Create  |
| `~`    | Update  |
| `-`    | Destroy |
| `-/+`  | Replace |

Always review the plan before applying infrastructure changes.

---

## 28. Resources and Terraform State

Terraform maintains state to track the relationship between configuration and managed infrastructure.

Conceptually:

```text
Terraform Configuration
        │
        ▼
       Plan
        │
        ▼
     Terraform State
        │
        ▼
Real Infrastructure
```

State allows Terraform to determine what infrastructure it is managing and what changes are required.

For example, Terraform can associate:

```text
aws_dynamodb_table.employee
```

with the actual AWS DynamoDB table represented in state.

Detailed state management and remote backends will be covered later in the learning path.

---

## 29. Resource Drift

Drift occurs when infrastructure changes outside Terraform.

For example:

```text
Terraform configuration
        │
        │ expects retention = 30
        ▼
CloudWatch Log Group
        │
        │ manually changed to 90
        ▼
Configuration ≠ Infrastructure
```

Terraform can detect differences during planning.

Example:

```bash
terraform plan
```

The plan may propose changing the resource back toward the configuration.

This is one reason infrastructure changes should preferably be performed through Terraform rather than manual console changes.

---

## 30. Resource Naming Best Practices

Use clear and consistent local names.

Good:

```hcl
resource "aws_lambda_function" "employee_api" {
  ...
}
```

Less descriptive:

```hcl
resource "aws_lambda_function" "function1" {
  ...
}
```

Recommended naming principles:

* use descriptive names
* use `snake_case`
* avoid unnecessary abbreviations
* keep names consistent across modules
* separate Terraform local names from actual cloud resource names
* avoid hardcoding environment-specific values when variables can be used

---

## 31. Resource Design Best Practices

Good resource definitions should be:

### Reusable

Use variables instead of unnecessary hardcoded values.

```hcl
name = var.table_name
```

### Readable

Keep resource blocks understandable.

### Modular

Group related infrastructure into modules when appropriate.

### Least privilege

For IAM resources, grant only the permissions required.

### Reviewable

Always inspect:

```bash
terraform plan
```

before applying changes.

### Consistent

Follow naming, tagging, and formatting conventions.

---

## 32. Common Resource Mistakes

### Hardcoding environment values

Avoid:

```hcl
name = "employee-dev-table"
```

inside a reusable module.

Prefer:

```hcl
name = var.table_name
```

### Overusing `depends_on`

Avoid adding explicit dependencies when Terraform can already infer them.

### Overusing lifecycle rules

Do not add:

```hcl
ignore_changes = all
```

without a strong reason.

### Using unclear resource names

Avoid:

```hcl
resource "aws_lambda_function" "x" {
  ...
}
```

Prefer:

```hcl
resource "aws_lambda_function" "employee_api" {
  ...
}
```

### Applying without reviewing the plan

Avoid blindly running:

```bash
terraform apply
```

Review:

```bash
terraform plan
```

first.

---

## 33. Resource Workflow

The typical Terraform resource workflow is:

```text
Write Resource
      │
      ▼
terraform fmt
      │
      ▼
terraform validate
      │
      ▼
terraform plan
      │
      ▼
Review Changes
      │
      ▼
terraform apply
      │
      ▼
Infrastructure Created/Updated
      │
      ▼
Terraform State Updated
```

For destructive changes, perform an additional review before applying.

---

## 34. Learned vs Implemented

### 📚 Learned

The following resource concepts were learned:

* Terraform resource blocks
* resource types
* local resource names
* resource arguments
* resource attributes
* resource references
* resource addressing
* resource creation
* resource updates
* resource replacement
* resource destruction
* lifecycle configuration
* `create_before_destroy`
* `prevent_destroy`
* `ignore_changes`
* `count`
* `for_each`
* resource dependencies
* resource providers
* resource drift
* resource behavior in `terraform plan`

### 🛠️ Implemented

The Employee Management API project already uses Terraform resources to define infrastructure such as:

* DynamoDB
* IAM roles
* IAM policies
* Lambda

Resources are organized through reusable Terraform modules.

Examples include:

```text
modules/dynamodb
modules/iam
modules/lambda
```

The project uses resource references to connect infrastructure components.

For example:

```hcl
role = aws_iam_role.lambda.arn
```

and:

```hcl
aws_dynamodb_table.employee.name
```

The project also uses:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

as part of the Terraform workflow.

### 🔮 Future Improvements

Future resource-related improvements can include:

* stronger lifecycle protection for critical resources
* standardized resource tagging
* more reusable resource modules
* additional AWS resources
* production-safe resource policies
* drift detection and remediation
* resource-level security improvements

These should be implemented only when required by the project rather than added purely for documentation.

---

## 35. Key Takeaways

Terraform resources are the core units of infrastructure managed by Terraform.

The fundamental syntax is:

```hcl
resource "<TYPE>" "<NAME>" {
  ...
}
```

Resources can:

* create infrastructure
* update infrastructure
* replace infrastructure
* destroy infrastructure
* expose attributes
* reference other resources
* participate in dependencies
* use lifecycle controls
* be created multiple times using `count` or `for_each`

A simplified mental model is:

```text
Terraform Configuration
          │
          ▼
       Resources
          │
          ▼
       Terraform
          │
          ▼
   Cloud Infrastructure
```

Understanding resources is essential before moving deeper into Terraform dependencies, lifecycle behavior, state management, and advanced infrastructure design.

> **Next stage:** Terraform Data Sources — learning how Terraform can read existing infrastructure and external information without creating those objects itself.
