# Terraform Fundamentals

## 1. Overview

Terraform is an Infrastructure as Code (IaC) tool used to define, provision, and manage infrastructure using configuration files.

Instead of manually creating infrastructure through the AWS Console, Terraform allows infrastructure to be described as code.

For example, an AWS DynamoDB table can be defined using Terraform:

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

Terraform can then create and manage this infrastructure consistently.

The overall concept is:

```text
Terraform Configuration
        │
        ▼
     Terraform
        │
        ▼
   AWS Provider
        │
        ▼
 AWS Infrastructure
```

Terraform is commonly used for:

* Infrastructure provisioning
* Infrastructure management
* Environment standardization
* Reusable infrastructure
* Infrastructure automation
* CI/CD infrastructure deployment

---

## 2. Infrastructure as Code

Infrastructure as Code means defining infrastructure using machine-readable configuration files instead of manually creating resources.

### Traditional Infrastructure Management

```text
Developer / Engineer
        │
        ▼
    AWS Console
        │
        ├── Create Lambda
        ├── Create DynamoDB
        ├── Create IAM Role
        └── Configure API Gateway
```

This approach can become difficult to reproduce and maintain.

### Infrastructure as Code

```text
Terraform Configuration
        │
        ▼
    Terraform
        │
        ▼
 AWS Infrastructure
```

The infrastructure configuration can be:

* Version controlled
* Reviewed
* Reused
* Automated
* Reproduced

This makes infrastructure easier to manage as the project grows.

---

## 3. Declarative Configuration

Terraform uses a **declarative configuration model**.

Declarative configuration describes **what the desired infrastructure should look like**, rather than providing step-by-step instructions for how to create it.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

The configuration describes the desired DynamoDB table.

Terraform determines the operations required to achieve that desired state.

Conceptually:

```text
Desired Configuration
        │
        ▼
     Terraform
        │
        ▼
Determine Required Changes
        │
        ▼
AWS Infrastructure
```

This is different from an imperative approach where the user would explicitly instruct the system:

```text
1. Create a DynamoDB table
2. Configure billing mode
3. Configure the partition key
4. Apply the configuration
```

Terraform instead describes the final desired configuration.

---

## 4. Terraform Providers

Terraform uses **providers** to communicate with external platforms and services.

A provider acts as the connection between Terraform and an infrastructure platform.

For AWS:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.40"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

The AWS provider allows Terraform to interact with AWS services.

Conceptually:

```text
Terraform
    │
    ▼
AWS Provider
    │
    ├── DynamoDB
    ├── Lambda
    ├── IAM
    ├── API Gateway
    ├── CloudWatch
    └── Other AWS Services
```

Providers are responsible for understanding how Terraform communicates with the target platform.

Examples of providers include:

* AWS
* Azure
* Google Cloud
* Kubernetes
* GitHub

Provider configuration is documented separately in:

`Terraform/providers.md`

---

## 5. Terraform Resources

A **resource** represents infrastructure that Terraform creates and manages.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

The general syntax is:

```hcl
resource "<RESOURCE_TYPE>" "<LOCAL_NAME>" {
  ...
}
```

In this example:

```text
aws_dynamodb_table
        │
        └── Resource Type

employee
        │
        └── Local Resource Name
```

Terraform identifies the resource as:

```text
aws_dynamodb_table.employee
```

Resources can represent:

* DynamoDB tables
* Lambda functions
* IAM roles
* IAM policies
* API Gateway resources
* CloudWatch resources
* VPCs
* S3 buckets
* And many other infrastructure objects

Resources are the primary building blocks of Terraform infrastructure.

Detailed resource documentation is available in:

`Terraform/resources.md`

---

## 6. Terraform Variables

Variables allow Terraform configurations to accept values dynamically instead of hardcoding them.

Example:

```hcl
variable "table_name" {
  description = "Name of the DynamoDB table"
  type        = string
}
```

The variable can then be used inside a resource:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

A value can be supplied through:

```hcl
table_name = "employee-dev-table"
```

Variables are useful for:

* Environment-specific configuration
* Reusable modules
* Avoiding hardcoded values
* Improving configuration flexibility

Common variable types include:

```text
string
number
bool
list
map
object
set
```

Detailed variable concepts are documented in:

`Terraform/variables.md`

---

## 7. Terraform State

Terraform state is Terraform's record of the infrastructure it manages.

At a high level:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
Terraform Configuration
        │
        ▼
     Terraform
        │
        ├───────────────┐
        ▼               ▼
 Terraform State   Actual Infrastructure
</pre>

Terraform uses state to keep track of resources that it manages and their known attributes.

For example, when Terraform creates a DynamoDB table, state stores information that allows Terraform to associate the Terraform resource:

```text
aws_dynamodb_table.employee
```

with the corresponding AWS infrastructure.

A local Terraform project commonly stores state in:

```text
terraform.tfstate
```

State helps Terraform determine whether infrastructure:

```text
Needs to be created
Needs to be updated
Needs to be replaced
Needs to be destroyed
Requires no change
```

### Important

Terraform state is important infrastructure data and should be handled carefully.

State should not be treated like normal application source code.

Topics such as:

* Remote state
* S3 backend
* State locking
* State security
* State migration
* State recovery

are covered separately in the **Terraform State and Remote Backend** stage.

---

## 8. Terraform Data Sources

A **data source** allows Terraform to read information about existing infrastructure or external information without creating that object as a Terraform resource.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

Terraform can then access the returned information:

```hcl
data.aws_caller_identity.current.account_id
```

The important difference is:

```text
Resource
    │
    └── Creates / manages infrastructure

Data Source
    │
    └── Reads existing information
```

### Resource vs Data Source

| Feature                | Resource | Data Source                            |
| ---------------------- | -------- | -------------------------------------- |
| Creates infrastructure | Yes      | No                                     |
| Manages infrastructure | Yes      | No                                     |
| Reads information      | Yes      | Yes                                    |
| Terraform lifecycle    | Managed  | Read-only from Terraform's perspective |

Data sources are useful when Terraform needs information that already exists outside the current configuration.

Detailed data source usage is documented in:

`Terraform/data-sources.md`

---

## 9. Terraform Outputs

Outputs expose useful information from Terraform configuration.

Example:

```hcl
output "dynamodb_table_name" {
  value = aws_dynamodb_table.employee.name
}
```

After applying the configuration, Terraform can display the output:

```text
dynamodb_table_name = "employee-dev-table"
```

Outputs can be used to:

* Display important infrastructure information
* Pass information between modules
* Expose resource attributes
* Support other Terraform configurations

Example:

```text
DynamoDB Module
      │
      │ table_arn
      ▼
IAM Module
```

Outputs are especially important when working with reusable Terraform modules.

Detailed output concepts are documented in:

`Terraform/outputs.md`

---

## 10. Terraform Modules

A Terraform module is a reusable collection of Terraform configuration.

Modules help organize infrastructure into logical components.

A simple module structure can look like:

```text
terraform/
│
├── modules/
│   ├── dynamodb/
│   ├── iam/
│   └── lambda/
│
└── dev/
```

A root module can call a child module:

```hcl
module "dynamodb" {
  source = "../modules/dynamodb"

  table_name = "employee-dev-table"
}
```

The module can expose values using outputs:

```hcl
output "table_arn" {
  value = aws_dynamodb_table.employee.arn
}
```

Another module can consume that output.

Conceptually:

```text
Root Module
     │
     ├── DynamoDB Module
     │        │
     │        └── table_arn
     │
     ├── IAM Module
     │
     └── Lambda Module
```

Modules provide:

* Reusability
* Consistency
* Better organization
* Separation of responsibilities
* Easier environment configuration

Modules were introduced progressively in the project and are documented in more detail in:

`Terraform/modules.md`

---

## 11. Configuration, State, and Infrastructure

One of the most important Terraform concepts is understanding the relationship between **configuration, state, and actual infrastructure**.

```text
        Terraform Configuration
                 │
                 │ Desired State
                 ▼
             Terraform
                 │
                 │ evaluates
                 ▼
        Terraform State
                 │
                 │ tracks
                 ▼
        Actual Infrastructure
```

### Terraform Configuration

The `.tf` files describe the desired infrastructure.

Example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = "employee-dev-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

### Terraform State

Terraform state records information about resources Terraform manages.

### Actual Infrastructure

Actual infrastructure is what currently exists in AWS.

Terraform uses the configuration, state, and information read from the provider to determine what changes are required.

The result can be:

```text
Create
Update
Replace
Destroy
No Change
```

This relationship is fundamental to understanding how Terraform works.

---

## 12. Desired State and Terraform Plan

Terraform follows a desired-state model.

The configuration represents the desired infrastructure.

When `terraform plan` is executed, Terraform evaluates the configuration and current known infrastructure information to determine the proposed changes.

Conceptually:

```text
Desired Configuration
        │
        ▼
 terraform plan
        │
        ▼
Evaluate Current State
        │
        ▼
Proposed Changes
        │
        ▼
Review
        │
        ▼
 terraform apply
        │
        ▼
Infrastructure Updated
```

For example:

```text
Configuration:

DynamoDB table should exist

Current infrastructure:

DynamoDB table does not exist

Terraform plan:

+ Create DynamoDB table
```

If the infrastructure already matches the configuration:

```text
No changes
Your infrastructure matches the configuration.
```

The plan-before-apply workflow provides an important review point before infrastructure changes are applied.

---

## 13. Terraform Project Structure

A basic Terraform project can be organized as:

```text
terraform/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

### `main.tf`

Usually contains the main infrastructure resources or module calls.

### `provider.tf`

Contains provider configuration.

### `variables.tf`

Defines input variables.

### `outputs.tf`

Defines output values.

### `terraform.tfvars`

Contains values for variables.

Example:

```hcl
table_name = "employee-dev-table"
```

As the project grows, the structure can evolve into reusable modules and environment-specific configurations.

---

## 14. Terraform Workflow

The standard Terraform workflow is:

```text
Write Configuration
        │
        ▼
terraform init
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
Review Plan
        │
        ▼
terraform apply
        │
        ▼
Infrastructure Updated
```

### 14.1 Write Configuration

Create or modify `.tf` files.

### 14.2 Initialize Terraform

```bash
terraform init
```

This initializes the working directory and installs required providers and modules.

### 14.3 Format Configuration

```bash
terraform fmt -recursive
```

This formats Terraform configuration consistently.

### 14.4 Validate Configuration

```bash
terraform validate
```

This checks whether the configuration is syntactically valid and internally consistent.

### 14.5 Generate a Plan

```bash
terraform plan
```

This shows the proposed infrastructure changes.

### 14.6 Apply Changes

```bash
terraform apply
```

This applies the approved infrastructure changes.

---

## 15. Terraform in the Employee Management API

Terraform is used to define infrastructure for the Serverless Employee Management API project.

The application architecture is based around AWS serverless services.

High-level architecture:

```text
Client
  │
  ▼
API Gateway
  │
  ▼
Lambda
  │
  ▼
DynamoDB
```

Supporting infrastructure includes services such as:

```text
IAM
CloudWatch
KMS
```

Terraform allows these infrastructure components to be defined as code.

The infrastructure can therefore be:

* Version controlled
* Reviewed
* Recreated
* Modified consistently
* Automated

---

## 16. Terraform Architecture Evolution

The Terraform project evolved progressively.

```text
Terraform Fundamentals
        │
        ▼
Project Structure
        │
        ▼
Providers & Resources
        │
        ▼
Variables
        │
        ▼
Outputs
        │
        ▼
Dependencies
        │
        ▼
Reusable Modules
        │
        ▼
Environment Separation
        │
        ▼
Remote State
        │
        ▼
CI/CD
        │
        ▼
Advanced Infrastructure Engineering
```

The goal is to understand Terraform fundamentals first and gradually introduce more advanced infrastructure engineering concepts.

This prevents advanced Terraform features from being introduced before the underlying concepts are understood.

---

## 17. Core Terraform Concepts

| Concept     | Purpose                                        |
| ----------- | ---------------------------------------------- |
| Provider    | Connects Terraform to an external platform     |
| Resource    | Creates and manages infrastructure             |
| Data Source | Reads existing information                     |
| Variable    | Provides configurable input values             |
| Output      | Exposes useful Terraform values                |
| Module      | Provides reusable infrastructure configuration |
| State       | Tracks infrastructure managed by Terraform     |
| Backend     | Defines where Terraform state is stored        |
| Plan        | Shows proposed infrastructure changes          |
| Apply       | Applies the proposed infrastructure changes    |

Backend and remote-state concepts are intentionally covered in detail in the **Terraform State and Remote Backend** stage.

---

## 18. Common Terraform Commands

### Initialize

```bash
terraform init
```

### Format

```bash
terraform fmt
```

For the complete project:

```bash
terraform fmt -recursive
```

### Validate

```bash
terraform validate
```

### Plan

```bash
terraform plan
```

### Apply

```bash
terraform apply
```

### Show Outputs

```bash
terraform output
```

### Show Current State

```bash
terraform show
```

These commands form the basic Terraform CLI workflow.

The complete CLI and workflow reference is documented in:

`Terraform/cli-and-workflow.md`

---

## 19. Key Terraform Principles

The main principles learned during the Terraform foundation stage are:

### Declarative Infrastructure

Describe the desired infrastructure instead of writing step-by-step provisioning instructions.

### Infrastructure as Code

Infrastructure is defined using version-controlled configuration files.

### Reusability

Modules and variables allow infrastructure configuration to be reused.

### Reproducibility

The same configuration can be used to create consistent infrastructure.

### Reviewability

Terraform plans allow infrastructure changes to be reviewed before applying them.

### Progressive Architecture

Terraform should be introduced progressively:

```text
Simple Configuration
        ↓
Variables
        ↓
Outputs
        ↓
Dependencies
        ↓
Modules
        ↓
Environments
        ↓
Remote State
        ↓
CI/CD
        ↓
Production Patterns
```

---

## 20. Learned vs Implemented

### 📚 Learned

The following Terraform fundamentals were learned:

* Infrastructure as Code
* Declarative configuration
* Terraform providers
* Resources
* Data sources
* Variables
* Outputs
* Terraform modules
* Terraform state
* Desired state
* Terraform plan
* Terraform workflow
* Terraform CLI commands
* Terraform project organization
* Basic dependency concepts
* Reusable infrastructure concepts
* Environment concepts

### 🛠️ Implemented

The Employee Management API project uses Terraform to define AWS infrastructure.

The project has progressively introduced:

* AWS provider configuration
* Terraform resources
* Variables
* Outputs
* Resource dependencies
* Reusable modules
* Environment-specific configuration

The Terraform configuration is maintained as code and can be reviewed and validated using the Terraform CLI workflow.

### 🔮 Future Improvements

The following areas are intentionally covered in later stages:

* Remote Terraform state
* S3 backend
* State locking
* State security
* State migration
* Advanced module design
* Terraform security scanning
* CI/CD integration
* Advanced lifecycle management
* Production infrastructure patterns

---

## 21. Key Takeaways

Terraform is a declarative Infrastructure as Code tool that allows infrastructure to be defined, reviewed, version-controlled, and managed through configuration.

The core Terraform concepts are:

```text
Provider
   ↓
Resources
   ↓
Variables
   ↓
Outputs
   ↓
Data Sources
   ↓
Modules
   ↓
State
   ↓
Plan
   ↓
Apply
```

The most important foundation is understanding how these concepts work together.

The Employee Management API project uses these concepts progressively rather than introducing every Terraform feature at once.

Understanding these fundamentals provides the foundation for more advanced topics such as:

* Reusable infrastructure
* Environment separation
* Remote state
* CI/CD
* Security
* Production infrastructure management

This completes the **Terraform Fundamentals** documentation stage and provides the conceptual foundation for the detailed Terraform reference pages that follow.
