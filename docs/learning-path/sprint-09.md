# Sprint 9 — Reusable Infrastructure

## Overview

Sprint 9 focused on making the Terraform infrastructure reusable across different environments.

A production-ready infrastructure project should not require completely separate Terraform implementations for development, testing and production.

Instead, reusable modules should define how infrastructure is created, while environment-specific configuration determines the values used by those modules.

---

## 1. Why Reusable Infrastructure?

Real-world applications commonly have multiple environments:

```text
Development
Testing
Production
```

Each environment may have different:

* Resource names.
* Configuration values.
* Environment variables.
* Scaling requirements.
* Tags.
* Infrastructure parameters.

However, the underlying infrastructure architecture should remain consistent.

---

## 2. Reusable Module Approach

The desired architecture is:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
              Reusable Modules
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
         DEV        TEST       PROD
          │          │          │
     Environment Environment Environment
       Values      Values      Values
</pre>


The module implementation remains the same.

Only the configuration values change.

---

## 3. Environment-Specific Configuration

A reusable module should not contain unnecessary environment-specific values.

For example:

```hcl
module "lambda" {

  source = "../../modules/lambda"

  function_name = var.lambda_function_name
  runtime       = var.lambda_runtime
  handler       = var.lambda_handler
}
```

The development environment can provide:

```hcl
lambda_function_name = "employee-api-dev"
```

A production environment could provide:

```hcl
lambda_function_name = "employee-api-prod"
```

The Lambda module itself does not need to be modified.

---

## 4. `terraform.tfvars`

Environment-specific values can be supplied using Terraform variable files.

For example:

```hcl
lambda_function_name = "employee-api-dev"

lambda_runtime = "python3.13"

lambda_handler = "app.lambda_handler"
```

This allows the Terraform configuration to separate:

```text
Infrastructure Logic
        ↓
Terraform Modules

Environment Values
        ↓
terraform.tfvars
```

---

## 5. Reusing the DynamoDB Module

The same DynamoDB module can be used for different environments.

Development:

```hcl
module "dynamodb" {

  source = "../../modules/dynamodb"

  table_name = "employee-dev-table"

  hash_key = "employee_id"
}
```

Production:

```hcl
module "dynamodb" {

  source = "../../modules/dynamodb"

  table_name = "employee-prod-table"

  hash_key = "employee_id"
}
```

The module implementation remains unchanged.

Only the input values are different.

---

## 6. DRY Principle

Reusable infrastructure follows the:

**DRY — Don't Repeat Yourself**

principle.

Instead of duplicating:

```text
Lambda resource
IAM resource
DynamoDB resource
```

for every environment, reusable modules can be created once.

The architecture becomes:

```text
Reusable Module
       ↓
Environment Configuration
       ↓
Environment Resources
```

---

## 7. Separation of Infrastructure Logic and Configuration

A good Terraform architecture separates:

### Infrastructure Logic

Defines **how** infrastructure is created.

```text
Terraform Modules
```

### Environment Configuration

Defines **which values** should be used.

```text
Variables
terraform.tfvars
Environment configuration
```

Conceptually:

```text
              Environment
              Configuration
                    │
                    ↓
              Module Inputs
                    │
                    ↓
             Reusable Module
                    │
                    ↓
             AWS Resources
```

---

## 8. Current Project Environment

The Employee Management API project initially focuses on the:

```text
dev
```

environment.

The infrastructure was structured so that reusable modules could later support additional environments without duplicating the infrastructure implementation.

This provides a foundation for future:

```text
dev
test
prod
```

configurations.

---

## 9. Benefits of Reusable Infrastructure

Reusable Terraform infrastructure provides several benefits.

### Consistency

Different environments follow the same infrastructure pattern.

### Maintainability

Infrastructure logic is maintained in one place.

### Reduced Duplication

The same resource definitions do not need to be copied.

### Scalability

Additional environments can be introduced more easily.

### Standardization

Infrastructure follows the same structure across environments.

---

## 10. Module Interface

The module interface should remain simple and expose only the configuration that needs to be controlled by the caller.

For example, a Lambda module can have:

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

This keeps the module reusable without exposing unnecessary implementation details.

---

## 11. Application to Employee Management API

The Employee Management API infrastructure was initially implemented for the development environment.

The reusable architecture separates:

```text
Reusable Infrastructure
        ↓
DynamoDB
IAM
Lambda
        ↓
Environment Configuration
        ↓
dev
```

The same module design can later be used for other environments by changing configuration values rather than copying the Terraform resource definitions.

---

## 12. Learned vs Implemented

### 📚 Learned

* Reusable Terraform infrastructure
* Environment-specific configuration
* `terraform.tfvars`
* DRY principle
* Environment separation
* Module reuse
* Module interfaces
* Infrastructure consistency
* Configuration vs infrastructure logic

### 🛠️ Implemented

The Employee Management API Terraform configuration was structured around reusable modules.

Environment-specific values were separated from reusable infrastructure logic.

The `dev` environment uses the reusable modules for infrastructure components such as:

* DynamoDB
* IAM
* Lambda

The architecture was designed so that additional environments can be introduced without duplicating the module implementation.

### 🔮 Future Improvements

Reusable infrastructure can be further improved by:

* Creating dedicated environment configurations for test and production.
* Introducing module versioning.
* Creating a shared internal module repository.
* Standardizing tags across environments.
* Adding environment-specific validation.
* Introducing remote state separation per environment.
* Improving module documentation.
* Adding CI validation for Terraform modules.

---

## 13. Key Takeaways

> Reusable Terraform modules allow infrastructure logic to be written once and reused across environments.

> Environment-specific values should be separated from reusable module implementation.

> The DRY principle helps prevent duplicated infrastructure code.

> Module inputs allow environments to provide different configuration values.

> A well-designed module can support dev, test and production without changing the underlying infrastructure logic.

Sprint 9 completed the **Dependencies, Modules and Reusable Infrastructure** stage and established the foundation for building a more scalable Terraform architecture.
