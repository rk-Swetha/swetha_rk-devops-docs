# Sprint 3 — AWS Provider & Resources

## Overview

Sprint 3 focused on understanding how Terraform communicates with AWS and how AWS infrastructure is represented using Terraform resources.

The two major concepts introduced were:

1. **Terraform providers**
2. **Terraform resources**

These concepts form the foundation of using Terraform to manage AWS infrastructure.

---

## 1. What is a Terraform Provider?

A Terraform provider is a plugin that allows Terraform to interact with an external platform, service, or API.

Terraform itself does not contain the implementation for every cloud provider.

Instead, providers allow Terraform to communicate with platforms such as:

* AWS
* Azure
* Google Cloud
* Kubernetes
* GitHub

For the Employee Management API, the **AWS provider** was used.

---

## 2. AWS Provider

The AWS provider allows Terraform to create and manage AWS resources.

A basic provider configuration is:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

The `region` specifies the AWS region in which regional resources are created.

---

## 3. Provider Version Constraints

Terraform provider requirements can be specified using the `terraform` block.

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.40"
    }
  }
}
```

The provider configuration and provider requirement serve different purposes.

### Provider requirement

```hcl
required_providers
```

defines which provider Terraform should use and the acceptable version constraint.

### Provider configuration

```hcl
provider "aws"
```

defines how that provider should be configured, such as the AWS region.

---

## 4. What is a Terraform Resource?

A Terraform resource represents an infrastructure object that Terraform manages.

General syntax:

```hcl
resource "<RESOURCE_TYPE>" "<NAME>" {
  # configuration
}
```

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

Here:

```text
aws_dynamodb_table
        ↓
Resource type

employee
        ↓
Terraform resource name
```

---

## 5. Resource Type vs Resource Name

In:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

`aws_dynamodb_table` is the **resource type**.

`employee` is the **local Terraform resource name**.

The local name is used to reference the resource elsewhere in Terraform.

For example:

```hcl
aws_dynamodb_table.employee.name
```

references the name attribute of that resource.

---

## 6. Provider vs Resource

These concepts should not be confused.

```text
Terraform
    │
    ▼
Provider
    │
    ▼
Resources
```

For this project:

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
    └── KMS
```

The **provider** enables Terraform to communicate with AWS.

The **resources** represent the individual AWS infrastructure objects being managed.

---

## 7. AWS Resources Used in the Project

The Employee Management API infrastructure eventually included several AWS services.

### DynamoDB

Used as the database for employee information.

Terraform resource example:

```hcl
resource "aws_dynamodb_table" "employee" {
  ...
}
```

### Lambda

Used to run the API application code without managing servers.

Terraform resource:

```hcl
aws_lambda_function
```

### API Gateway

Used as the API entry point for clients.

Terraform resources from the API Gateway module were used to expose the Lambda-based API.

### IAM

Used to define permissions and roles required by AWS services.

The Lambda execution role and associated permissions were managed through Terraform.

### KMS

Used for encryption-related infrastructure.

The project included a KMS module to manage encryption configuration.

### CloudWatch

Used for logging and observability of AWS resources such as Lambda.

---

## 8. Resource Attributes

Resources contain arguments and attributes.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

Arguments configure the resource.

Attributes can expose information about the resource after Terraform creates it.

For example:

```hcl
aws_dynamodb_table.employee.name
```

can be used elsewhere in the configuration.

---

## 9. Application to Employee Management API

The AWS provider became the connection between Terraform and the AWS infrastructure required by the project.

The infrastructure direction was:

```text
Employee Management API
        │
        ▼
     Terraform
        │
        ▼
   AWS Provider
        │
        ├── API Gateway
        ├── Lambda
        ├── DynamoDB
        ├── IAM
        ├── KMS
        └── CloudWatch
```

As the project progressed, these resources were reorganized into reusable modules.

---

## 10. Learned vs Implemented

### 📚 Learned

* Terraform providers
* AWS provider
* Provider version constraints
* Terraform resources
* Resource types
* Resource names
* Resource arguments and attributes
* Difference between providers and resources

### 🛠️ Implemented

The AWS provider was used to manage the AWS infrastructure for the Employee Management API.

Terraform resources were used for services including Lambda, DynamoDB, IAM, API Gateway, and KMS.

CloudWatch was also part of the project's AWS infrastructure and observability design.

### 🔮 Future Improvements

The project can later expand to additional AWS services when required by the application architecture.

---

## 11. Key Takeaways

> Providers allow Terraform to communicate with external platforms such as AWS.

> Resources represent infrastructure objects managed by Terraform.

> The provider and resource are different concepts.

> Provider version constraints help control which provider versions Terraform can use.

> Resource references allow information from one resource to be used by another part of the Terraform configuration.

The concepts learned in this sprint became the foundation for variables, dependencies, modules, and environment-specific infrastructure introduced in later sprints.
