# Terraform Data Sources

## 1. Overview

Terraform resources are used to create and manage infrastructure.

Terraform also needs a way to **read information about infrastructure that already exists**.

This is where **data sources** are used.

A data source allows Terraform to retrieve information from an external system or existing infrastructure without managing the lifecycle of that object.

For example, Terraform can use a data source to:

* Read an existing AWS account ID
* Find an existing VPC
* Look up an existing subnet
* Retrieve information about an IAM policy
* Find an existing AMI
* Read information about the current AWS region
* Use existing infrastructure as input for newly created resources

A simplified model is:

```text
Resource
   │
   ├── Creates infrastructure
   ├── Updates infrastructure
   └── Deletes infrastructure

Data Source
   │
   └── Reads existing information
```

---

## 2. What Is a Terraform Data Source?

A **data source** is a read-only interface that allows Terraform to retrieve information from a provider or external system.

Data sources are declared using a `data` block.

Basic syntax:

```hcl
data "<DATA_SOURCE_TYPE>" "<LOCAL_NAME>" {
  # arguments
}
```

Example:

```hcl
data "aws_caller_identity" "current" {}
```

Terraform reads information about the AWS identity being used.

The data source can then be referenced elsewhere:

```hcl
data.aws_caller_identity.current.account_id
```

---

## 3. Data Source vs Resource

The most important distinction is:

```text
Resource
→ Terraform manages the object

Data Source
→ Terraform reads the object
```

Example resource:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

Terraform is responsible for creating and managing this DynamoDB table.

Example data source:

```hcl
data "aws_caller_identity" "current" {}
```

Terraform does not create the AWS account identity.

It only reads information about the identity.

### Comparison

| Feature                   | Resource                   | Data Source      |
| ------------------------- | -------------------------- | ---------------- |
| Purpose                   | Manage infrastructure      | Read information |
| Creates objects           | Yes                        | No               |
| Updates objects           | Yes                        | No               |
| Deletes objects           | Yes                        | No               |
| Read existing information | Yes, as part of management | Yes              |
| Lifecycle management      | Yes                        | No               |
| Syntax                    | `resource`                 | `data`           |

---

## 4. Data Source Syntax

The general structure is:

```hcl
data "<TYPE>" "<NAME>" {
  argument = value
}
```

There are two important identifiers:

### Data Source Type

The type determines what Terraform is reading.

Example:

```hcl
aws_caller_identity
```

### Local Name

The local name identifies the data source inside the Terraform configuration.

Example:

```hcl
current
```

Complete example:

```hcl
data "aws_caller_identity" "current" {}
```

The full Terraform address is:

```text
data.aws_caller_identity.current
```

---

## 5. Data Source Arguments

Data sources can accept arguments that determine what information Terraform should retrieve.

Example:

```hcl
data "aws_region" "current" {}
```

Another example:

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*"]
  }
}
```

Here:

* `most_recent` controls which AMI should be selected
* `owners` restricts the search
* `filter` specifies matching criteria

Arguments are used to tell the provider **what information to look up**.

---

## 6. Data Source Attributes

After Terraform reads a data source, the provider exposes attributes containing the retrieved information.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

One available attribute is:

```hcl
data.aws_caller_identity.current.account_id
```

Another example:

```hcl
data "aws_region" "current" {}
```

The region can be accessed using:

```hcl
data.aws_region.current.name
```

The available attributes depend on the specific data source type.

---

## 7. Referencing Data Sources

Data sources use the following reference format:

```text
data.<TYPE>.<NAME>.<ATTRIBUTE>
```

Example:

```hcl
data.aws_region.current.name
```

Breaking it down:

```text
data
 │
 └── aws_region
       │
       └── current
             │
             └── name
```

This value can be passed to another resource.

Example:

```hcl
resource "aws_cloudwatch_log_group" "application" {
  name = "/aws/lambda/${data.aws_region.current.name}/application"
}
```

---

## 8. Data Sources as Inputs to Resources

One of the most useful patterns is:

```text
Existing Infrastructure
        │
        ▼
   Data Source
        │
        ▼
   Retrieved Value
        │
        ▼
      Resource
```

For example:

```hcl
data "aws_caller_identity" "current" {}
```

The account ID can be used in an IAM policy:

```hcl
resource "aws_iam_policy" "example" {
  name = "example-policy"

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [
      {
        Effect = "Allow"

        Resource = "arn:aws:s3:::example-${data.aws_caller_identity.current.account_id}/*"

        Action = [
          "s3:GetObject"
        ]
      }
    ]
  })
}
```

Terraform first reads the account ID and then uses that value while constructing the policy.

---

## 9. Data Sources and Dependencies

Data sources can participate in Terraform's dependency graph.

For example:

```hcl
data "aws_caller_identity" "current" {}

resource "aws_iam_policy" "example" {
  policy = jsonencode({
    Statement = [
      {
        Resource = "arn:aws:s3:::example-${data.aws_caller_identity.current.account_id}/*"
      }
    ]
  })
}
```

The resource references the data source.

Therefore Terraform understands that the data source value is required before the resource configuration can be evaluated.

Conceptually:

```text
AWS Identity
     │
     ▼
Data Source
     │
     ▼
Account ID
     │
     ▼
IAM Policy
```

Detailed dependency graph behavior is covered separately in the dependency and lifecycle documentation.

---

## 10. Common AWS Data Sources

The AWS provider provides many useful data sources.

Some common examples include:

```text
aws_caller_identity
aws_region
aws_availability_zones
aws_vpc
aws_subnet
aws_security_group
aws_ami
aws_iam_policy
aws_iam_role
aws_canonical_user_id
```

The exact arguments and attributes depend on the individual data source.

Always check the provider documentation for the specific data source being used.

---

## 11. AWS Caller Identity

The `aws_caller_identity` data source retrieves information about the AWS identity Terraform is currently using.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

Example reference:

```hcl
data.aws_caller_identity.current.account_id
```

This is useful when infrastructure needs the AWS account ID without hardcoding it.

Instead of:

```hcl
account_id = "123456789012"
```

Terraform can dynamically obtain it:

```hcl
data.aws_caller_identity.current.account_id
```

This improves portability between AWS accounts.

---

## 12. AWS Region

Terraform can also retrieve information about the current AWS region.

Example:

```hcl
data "aws_region" "current" {}
```

Reference:

```hcl
data.aws_region.current.name
```

Example:

```hcl
output "current_region" {
  value = data.aws_region.current.name
}
```

This avoids hardcoding the region in places where the current provider region is already the source of truth.

---

## 13. AWS Availability Zones

The `aws_availability_zones` data source can retrieve available Availability Zones.

Example:

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}
```

The retrieved information can then be used by resources.

For example:

```hcl
availability_zone = data.aws_availability_zones.available.names[0]
```

This allows Terraform to dynamically determine available zones instead of manually hardcoding them.

---

## 14. Finding an Existing VPC

Data sources are useful when infrastructure needs to use an existing VPC.

Example:

```hcl
data "aws_vpc" "existing" {
  id = "vpc-12345678"
}
```

Terraform reads the VPC information.

It does not create or delete the VPC.

The VPC ID can then be passed to another resource:

```hcl
resource "aws_subnet" "application" {
  vpc_id = data.aws_vpc.existing.id
}
```

The relationship becomes:

```text
Existing VPC
     │
     ▼
aws_vpc Data Source
     │
     ▼
VPC ID
     │
     ▼
Subnet Resource
```

---

## 15. Finding an Existing Subnet

An existing subnet can also be retrieved.

Example:

```hcl
data "aws_subnet" "existing" {
  id = "subnet-12345678"
}
```

The subnet information can then be consumed by another resource.

For example:

```hcl
subnet_id = data.aws_subnet.existing.id
```

This is useful when Terraform-managed infrastructure must be deployed into networking infrastructure managed elsewhere.

---

## 16. Filtering Data Sources

Some data sources support filtering.

Filtering is useful when the exact resource ID is not known.

Example:

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```

Terraform searches for matching objects and retrieves the selected result.

Filtering can make configurations more dynamic.

However, filters should be specific enough to avoid accidentally selecting the wrong object.

---

## 17. Data Sources and Existing Infrastructure

A common real-world architecture is:

```text
Existing Infrastructure
        │
        ▼
    Data Sources
        │
        ▼
Terraform-managed Infrastructure
```

For example:

```text
Existing VPC
Existing Subnets
Existing IAM Resources
        │
        ▼
     Terraform
        │
        ▼
New Application Resources
```

This is particularly useful in organizations where infrastructure ownership is divided between teams.

For example:

```text
Networking Team
       │
       ├── VPC
       └── Subnets
              │
              ▼
        Data Sources
              │
              ▼
     Application Team
              │
              ├── Lambda
              └── API Gateway
```

Terraform can consume information about infrastructure without taking ownership of it.

---

## 18. Data Sources Inside Modules

Data sources can also be defined inside Terraform modules.

Example:

```text
terraform/
├── modules/
│   └── lambda/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
└── environments/
    └── dev/
        ├── main.tf
        └── terraform.tfvars
```

A module can contain:

```hcl
data "aws_region" "current" {}
```

and use:

```hcl
data.aws_region.current.name
```

inside its resources.

However, module interfaces should remain clear.

If a value is better supplied by the root module, it may be preferable to pass it as a variable instead of making the child module perform unnecessary lookups.

---

## 19. Data Sources vs Variables

Variables and data sources solve different problems.

### Variable

A variable receives a value from the caller.

```hcl
variable "environment" {
  type = string
}
```

Usage:

```hcl
environment = "dev"
```

### Data Source

A data source retrieves information from an external system.

```hcl
data "aws_region" "current" {}
```

Usage:

```hcl
data.aws_region.current.name
```

Conceptually:

```text
Variable
   │
   └── Input supplied by user/configuration

Data Source
   │
   └── Information retrieved from provider
```

---

## 20. Data Sources vs Locals

Local values are calculated from values already available to Terraform.

Example:

```hcl
locals {
  function_name = "employee-api-${var.environment}"
}
```

A data source, on the other hand, retrieves information externally.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

Therefore:

```text
Variable → receives input
Data Source → reads external information
Local → calculates/reuses values
```

These mechanisms can also work together.

Example:

```hcl
locals {
  bucket_name = "application-${var.environment}-${data.aws_caller_identity.current.account_id}"
}
```

---

## 21. Data Sources and Plan

Terraform evaluates data sources as part of its normal workflow.

During:

```bash
terraform plan
```

Terraform determines which data sources need to be read and how their values affect the proposed infrastructure changes.

Example:

```text
terraform plan
      │
      ▼
Read Data Sources
      │
      ▼
Evaluate Configuration
      │
      ▼
Build Dependency Graph
      │
      ▼
Calculate Proposed Changes
```

If a data source cannot be resolved, Terraform may fail during planning.

---

## 22. Data Sources and Apply

Data sources may be read during planning or during apply depending on whether their required inputs are already known.

For example, if a data source depends on another resource whose value is unknown until apply, Terraform may defer the data source lookup.

Conceptually:

```text
Known inputs
     │
     ▼
Data Source
     │
     ▼
Read during planning
```

Whereas:

```text
Resource creation
     │
     ▼
Value becomes known
     │
     ▼
Data Source lookup
     │
     ▼
Continue evaluation
```

This is another reason why understanding dependencies is important.

---

## 23. Data Sources and State

Data source information can be represented in Terraform state as part of Terraform's knowledge about the configuration.

However, data sources are not managed in the same way as resources.

A resource represents an object Terraform manages.

A data source represents information Terraform has read.

High-level model:

```text
Terraform State
│
├── Managed Resources
│
└── Data Source Information
```

Detailed state management and remote state are covered in later documentation.

---

## 24. Data Sources and Drift

Data sources behave differently from managed resources when external infrastructure changes.

Suppose Terraform reads an existing VPC:

```hcl
data "aws_vpc" "existing" {
  id = "vpc-12345678"
}
```

If the VPC's attributes change outside Terraform, the data source can retrieve the current information again.

Terraform is not attempting to restore the VPC to a previously declared configuration because it does not manage that VPC.

This is a key difference between:

```text
Managed Resource
→ Terraform attempts to manage desired state

Data Source
→ Terraform reads current external information
```

---

## 25. Data Sources in the Employee Management API

The Employee Management API currently focuses on infrastructure managed through Terraform modules.

The project uses resources such as:

```text
API Gateway
Lambda
DynamoDB
IAM
CloudWatch
```

Data sources become useful when the project needs to consume information that already exists outside the Terraform-managed resources.

For example, the project could retrieve:

```text
AWS Account ID
AWS Region
Existing IAM resources
Existing networking resources
```

A simple example is:

```hcl
data "aws_caller_identity" "current" {}
```

The account ID can then be used when constructing resource-specific identifiers or policies.

Another example:

```hcl
data "aws_region" "current" {}
```

The region can be used dynamically in configuration or outputs.

---

## 26. Example: Account-Aware Naming

Instead of hardcoding an AWS account ID:

```hcl
locals {
  account_id = "123456789012"
}
```

Terraform can retrieve it dynamically:

```hcl
data "aws_caller_identity" "current" {}

locals {
  account_id = data.aws_caller_identity.current.account_id
}
```

The local value can then be reused:

```hcl
resource "aws_iam_policy" "application" {
  name = "employee-api-policy-${local.account_id}"

  # policy configuration
}
```

This makes the configuration more portable.

---

## 27. Example: Dynamic Region Information

Example:

```hcl
data "aws_region" "current" {}

output "deployment_region" {
  value = data.aws_region.current.name
}
```

The output reflects the AWS region configured for the provider.

This avoids maintaining a duplicate hardcoded region value.

---

## 28. When to Use Data Sources

Use a data source when:

* The object already exists.
* Terraform should not create the object.
* Terraform should not manage the object's lifecycle.
* Another system owns the object.
* Terraform needs information from an external system.
* A resource needs information that can be dynamically looked up.
* Hardcoding the value would reduce portability.

Example:

```text
Need existing VPC information
        ↓
Use data source
```

---

## 29. When Not to Use Data Sources

Do not use a data source simply because it is possible.

If Terraform should create and manage an object, use a resource.

Incorrect approach:

```hcl
data "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

if Terraform is actually responsible for creating and managing that table.

Correct approach:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"

  # configuration
}
```

Use the appropriate abstraction based on ownership.

---

## 30. Common Data Source Mistakes

### Mistake 1 — Confusing Resources and Data Sources

Using a data source when Terraform should manage the object.

### Mistake 2 — Hardcoding Dynamic Information

Example:

```hcl
account_id = "123456789012"
```

when the account ID can safely be retrieved dynamically.

### Mistake 3 — Overly Broad Filters

A filter that matches multiple objects may result in unexpected behavior.

### Mistake 4 — Assuming Every Resource Has a Data Source

Data sources depend on provider capabilities.

Not every resource type necessarily has an equivalent data source.

### Mistake 5 — Using Data Sources to Avoid Proper Module Inputs

Sometimes it is cleaner for the root module to provide a value to a child module rather than having the child module perform an unnecessary lookup.

---

## 31. Data Source Best Practices

### 31.1 Use Data Sources for External Information

Use them when Terraform needs information it does not own.

### 31.2 Prefer Dynamic Lookups Over Hardcoded Values

When appropriate, dynamically retrieve values such as:

```text
Account ID
Region
Availability Zones
Existing Resource IDs
```

### 31.3 Keep Filters Specific

When using filters, make the selection criteria predictable.

### 31.4 Respect Infrastructure Ownership

Do not use a data source as a substitute for resource management when Terraform should own the infrastructure.

### 31.5 Keep Module Interfaces Clear

Avoid unnecessary data lookups inside modules when values can be cleanly supplied as variables.

### 31.6 Document External Dependencies

If a module depends on infrastructure managed elsewhere, document that dependency clearly.

---

## 32. Practical Workflow

A typical workflow for using a data source is:

```text
1. Identify required external information
            ↓
2. Find the appropriate provider data source
            ↓
3. Declare the data block
            ↓
4. Configure lookup arguments
            ↓
5. Reference the returned attributes
            ↓
6. Use the values in resources/modules/outputs
            ↓
7. Run terraform fmt
            ↓
8. Run terraform validate
            ↓
9. Run terraform plan
```

Example:

```hcl
data "aws_caller_identity" "current" {}

resource "aws_iam_policy" "application" {
  name = "employee-api-${data.aws_caller_identity.current.account_id}"
}
```

---

## 33. Data Sources and Terraform Architecture

Data sources fit into Terraform architecture like this:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                    Terraform Configuration
                             │
              ┌──────────────┴──────────────┐
              │                             │
          Resources                    Data Sources
              │                             │
              ▼                             ▼
     Managed Infrastructure         External Information
              │                             │
              └──────────────┬──────────────┘
                             │
                             ▼
                       Terraform Graph
                             │
                             ▼
                         Plan / Apply
</pre>

Resources and data sources can work together to build complete infrastructure configurations.

---

## 34. Learned vs Implemented

### 📚 Learned

This documentation covered:

* What Terraform data sources are
* Data source syntax
* Data source types and local names
* Data source arguments
* Data source attributes
* Data source references
* Resources vs data sources
* Variables vs data sources
* Locals vs data sources
* AWS data sources
* Filtering
* Existing infrastructure lookups
* Data sources inside modules
* Data sources and dependencies
* Data sources and Terraform plan/apply
* Data sources and state
* Data sources and drift
* Data source best practices
* Common mistakes

### 🛠️ Implemented

For the current Employee Management API project:

* Terraform resources remain responsible for infrastructure that the project manages.
* Data sources can be introduced where the project needs information from existing AWS infrastructure.
* `aws_caller_identity` and `aws_region` are suitable examples for dynamically obtaining AWS account and region information.
* Data sources are documented as read-only lookups rather than infrastructure ownership mechanisms.

### 🔮 Future Improvements

As the project becomes more production-oriented, data sources may be used for:

* Existing VPCs
* Existing subnets
* Existing security groups
* Existing IAM resources
* Existing AWS networking infrastructure
* Cross-environment infrastructure lookups

These should only be introduced when they represent genuine infrastructure dependencies.

---

## 35. Key Takeaways

The most important concepts from Terraform Data Sources are:

```text
Resource
→ Terraform manages infrastructure

Data Source
→ Terraform reads existing information
```

A data source is declared using:

```hcl
data "<TYPE>" "<NAME>" {
  # arguments
}
```

It is referenced using:

```text
data.<TYPE>.<NAME>.<ATTRIBUTE>
```

For example:

```hcl
data "aws_caller_identity" "current" {}

data.aws_caller_identity.current.account_id
```

The key design principle is:

> **Use resources for infrastructure Terraform owns, and data sources for information Terraform needs but does not own.**

Data sources therefore complement resources rather than replacing them.

With resources, configuration language, and data sources now documented, the Terraform documentation is building a complete foundation for understanding how Terraform defines, reads, and manages infrastructure.

The next topic is **Terraform Variables**, which explains how configuration values are parameterized and passed into reusable infrastructure.
