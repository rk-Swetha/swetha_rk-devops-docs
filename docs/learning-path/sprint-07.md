# Sprint 7 — Terraform Modules

## Overview

Sprint 7 focused on Terraform modules.

As the Employee Management API infrastructure became larger, keeping all Terraform resources in a single configuration would make the project difficult to maintain and reuse.

Terraform modules provide a way to group related infrastructure resources together and reuse them across different parts of the project.

The project introduced modules for infrastructure components such as DynamoDB, IAM and Lambda.

---

## 1. What are Terraform Modules?

A Terraform module is a collection of Terraform configuration files that manages a specific set of infrastructure resources.

A module can contain:

```text
main.tf
variables.tf
outputs.tf
```

The module can then be called from another Terraform configuration.

Conceptually:

```text
Root Module
     │
     ├── DynamoDB Module
     │
     ├── IAM Module
     │
     └── Lambda Module
```

---

## 2. Root Module vs Child Modules

Terraform configurations have a root module.

The root module is the main configuration from which Terraform commands are executed.

Child modules are reusable modules called by the root module.

For example:

```text
Terraform Configuration
        │
        ↓
    Root Module
        │
        ├── DynamoDB Module
        ├── IAM Module
        └── Lambda Module
```

---

## 3. Why Modules Are Useful

Modules help to:

* Organize Terraform code.
* Reduce duplication.
* Separate infrastructure responsibilities.
* Improve maintainability.
* Improve reusability.
* Simplify complex Terraform configurations.
* Support multiple environments.

Without modules, the same infrastructure logic may need to be duplicated.

For example:

```text
dev/
    lambda.tf
    iam.tf
    dynamodb.tf

prod/
    lambda.tf
    iam.tf
    dynamodb.tf
```

This can become difficult to maintain.

---

## 4. Module Structure

The Employee Management API Terraform project follows a modular structure similar to:

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
    └── dev/
        ├── main.tf
        ├── variables.tf
        ├── terraform.tfvars
        └── outputs.tf
```

The exact structure can evolve as additional infrastructure components are introduced.

---

## 5. DynamoDB Module

The DynamoDB module is responsible for managing DynamoDB-related infrastructure.

Responsibilities can include:

* Creating the DynamoDB table.
* Configuring the partition key.
* Configuring table properties.
* Managing table-specific settings.
* Applying appropriate tags.

The root configuration can call the module using:

```hcl
module "dynamodb" {

  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"
  hash_key   = "employee_id"
}
```

The module handles the actual resource implementation.

---

## 6. IAM Module

The IAM module manages identity and access-related infrastructure.

Responsibilities can include:

* Creating the Lambda execution role.
* Creating IAM policies.
* Attaching policies.
* Providing role information to other modules.

Example:

```hcl
module "iam" {

  source = "../../modules/iam"

  role_name = "employee-api-dev-role"
}
```

---

## 7. Lambda Module

The Lambda module manages the AWS Lambda function.

Responsibilities can include:

* Creating the Lambda function.
* Configuring the runtime.
* Configuring the handler.
* Configuring the deployment package.
* Associating the IAM role.
* Configuring environment variables.

Example:

```hcl
module "lambda" {

  source = "../../modules/lambda"

  function_name = "employee-api-dev"
  runtime       = "python3.13"
  handler       = "app.lambda_handler"
}
```

---

## 8. Single Responsibility

Each Terraform module should ideally have a clear responsibility.

For example:

```text
DynamoDB Module
       ↓
Database infrastructure

IAM Module
       ↓
Identity and permissions

Lambda Module
       ↓
Application compute
```

This makes it easier to understand and modify infrastructure.

---

## 9. Modules and Reusability

The major benefit of modules is reuse.

Instead of creating separate implementations for each environment:

```text
Dev Lambda
Test Lambda
Prod Lambda
```

the project can use:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
Reusable Lambda Module
          │
     ┌────┼────┐
     ↓    ↓    ↓
    Dev  Test Prod
</pre>


Only the configuration values need to change.

---

## 10. Application to Employee Management API

The Employee Management API Terraform project was organized around reusable infrastructure components.

The major modules include:

```text
DynamoDB Module
IAM Module
Lambda Module
```

These modules separate infrastructure responsibilities and provide a foundation for further expansion.

Additional infrastructure such as API Gateway can also be separated into its own module as the project evolves.

---

## 11. Learned vs Implemented

### 📚 Learned

* Terraform modules
* Root modules
* Child modules
* Module structure
* Module source
* Module responsibilities
* Single Responsibility Principle
* Infrastructure reusability
* DRY principle

### 🛠️ Implemented

The Employee Management API infrastructure was organized into reusable modules.

The infrastructure responsibilities were separated into:

* DynamoDB
* IAM
* Lambda

The root configuration calls these modules instead of directly managing every resource in one place.

### 🔮 Future Improvements

Modules can be further improved by:

* Creating additional modules for API Gateway.
* Creating modules for CloudWatch resources.
* Adding module documentation.
* Introducing module versioning.
* Improving module validation.
* Creating a standard module structure.
* Reusing modules across multiple environments.

---

## 12. Key Takeaways

> Terraform modules group related infrastructure resources together.

> Modules help reduce duplication and improve maintainability.

> A good module should have a clear responsibility.

> Modules allow infrastructure logic to be reused across environments.

> Terraform modules are an important foundation for scalable infrastructure-as-code.

Sprint 7 established the reusable module structure that would be refined further using **module inputs and outputs** in Sprint 8.
