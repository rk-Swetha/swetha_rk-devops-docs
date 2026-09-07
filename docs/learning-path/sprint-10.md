# Sprint 10 — Terraform Environments

## Overview

Real-world applications rarely have only one infrastructure environment.

A typical application may have:

```text
Development
Testing
Staging
Production
```

Each environment may use the same infrastructure architecture while having different:

* resource names
* instance sizes
* scaling requirements
* configuration values
* AWS accounts or regions
* security settings
* deployment policies

Terraform allows us to organize infrastructure so that the underlying infrastructure logic can be reused while environment-specific configuration remains separate.

For the Employee Management API, the desired architecture is:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                    Reusable Modules
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
         dev              test             prod
          │                │                │
          ▼                ▼                ▼
    Dev Resources    Test Resources    Prod Resources
</pre>

This separation helps prevent accidental changes to one environment from affecting another.

---

## 1. What is a Terraform Environment?

A Terraform environment represents a specific deployment context.

For example:

```text
dev
test
prod
```

Each environment can have its own:

* Terraform configuration
* variable values
* state
* resource naming
* infrastructure settings

The infrastructure logic can still come from the same reusable modules.

For example:

```text
modules/
└── dynamodb/

environments/
├── dev/
├── test/
└── prod/
```

The same DynamoDB module can be used by all environments.

---

## 2. Why Multiple Environments Are Needed

Consider the Employee Management API.

Development may use:

```text
employee-dev-table
employee-api-dev
```

Production may use:

```text
employee-prod-table
employee-api-prod
```

The infrastructure type is the same, but the actual resources must remain separate.

This provides isolation:

```text
DEV
employee-dev-table
       │
       │
       └── Development data

PROD
employee-prod-table
       │
       │
       └── Production data
```

Development testing should never accidentally operate against production infrastructure.

---

## 3. Environment Separation

A common approach is to create separate environment directories.

Example:

```text
terraform/
│
├── modules/
│   ├── dynamodb/
│   ├── iam/
│   └── lambda/
│
└── environments/
    ├── dev/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars
    │
    ├── test/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars
    │
    └── prod/
        ├── main.tf
        ├── variables.tf
        └── terraform.tfvars
```

Each environment becomes a separate Terraform root module.

---

## 4. Environment Configuration

The environment configuration calls the reusable modules.

For example:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = var.table_name
  hash_key   = var.hash_key
}
```

The module itself doesn't know whether it is being used for development or production.

The environment supplies the values.

For development:

```text
table_name = employee-dev-table
```

For production:

```text
table_name = employee-prod-table
```

The module remains unchanged.

---

## 5. Environment-Specific Variables

Variables allow each environment to provide different values.

Example:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
}
```

The environment can provide:

```hcl
environment = "dev"
```

or:

```hcl
environment = "prod"
```

Other environment-specific values can include:

```hcl
variable "table_name" {
  description = "DynamoDB table name"
  type        = string
}

variable "function_name" {
  description = "Lambda function name"
  type        = string
}
```

This allows the same module logic to be reused with different configurations.

---

## 6. `.tfvars` Files

Terraform supports `.tfvars` files for providing variable values.

For example:

```text
environments/
└── dev/
    └── terraform.tfvars
```

The file could contain:

```hcl
environment  = "dev"
table_name   = "employee-dev-table"
function_name = "employee-api-dev"
```

Production could have:

```text
environments/
└── prod/
    └── terraform.tfvars
```

with:

```hcl
environment  = "prod"
table_name   = "employee-prod-table"
function_name = "employee-api-prod"
```

The module code does not need to change.

---

## 7. Environment Workflow

Terraform commands are executed from the appropriate environment directory.

For development:

```bash
cd terraform/environments/dev
```

Then:

```bash
terraform init
terraform validate
terraform plan
```

For production:

```bash
cd terraform/environments/prod
```

Then:

```bash
terraform init
terraform validate
terraform plan
```

Each directory represents an independent Terraform root module.

---

## 8. Separate State Per Environment

Environment separation is especially important for Terraform state.

Ideally:

```text
DEV
   ↓
Dev State

TEST
   ↓
Test State

PROD
   ↓
Prod State
```

This prevents one environment's state from being mixed with another.

For example:

```text
dev state
    │
    └── employee-dev-table

prod state
    │
    └── employee-prod-table
```

State management will be covered in greater detail in the next batch.

---

## 9. Environment-Specific Naming

Resource names should clearly identify their environment.

For example:

```text
employee-dev-table
employee-test-table
employee-prod-table
```

Similarly:

```text
employee-api-dev
employee-api-test
employee-api-prod
```

This makes AWS resources easier to identify.

A simplified naming pattern is:

```text
<application>-<environment>-<resource>
```

For example:

```text
employee-dev-table
```

can be interpreted as:

```text
employee
   +
dev
   +
table
```

Consistent naming becomes increasingly important when many resources exist.

---

## 10. Environment-Specific Configuration

Not every environment needs identical settings.

For example:

```text
Development
- smaller resources
- lower cost
- easier experimentation

Production
- stronger security
- higher availability
- stricter access controls
- monitoring
- backups
```

The reusable module can support these differences through variables.

For example:

```hcl
variable "enable_point_in_time_recovery" {
  description = "Enable DynamoDB point-in-time recovery"
  type        = bool
}
```

Development could use:

```hcl
enable_point_in_time_recovery = false
```

while production could use:

```hcl
enable_point_in_time_recovery = true
```

The underlying module remains reusable.

---

## 11. Same Module, Different Configuration

Consider the DynamoDB module:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = var.table_name
  hash_key   = "employee_id"
}
```

Development:

```text
table_name = employee-dev-table
```

Production:

```text
table_name = employee-prod-table
```

The architecture becomes:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                    DynamoDB Module
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
             DEV                       PROD
              │                         │
    employee-dev-table       employee-prod-table
</pre>

This is the primary advantage of separating modules from environment coniguration.

---

## 12. Environment Isolation

Environment separation is not only about naming.

It should also provide isolation of infrastructure.

A mature setup may use:

```text
Development → AWS Account A
Testing     → AWS Account B
Production  → AWS Account C
```

or separate accounts with appropriate organizational controls.

This provides stronger protection against accidental production changes.

For the current Employee Management API project, a single development environment is sufficient while the infrastructure is being built.

Multi-environment AWS account architecture can be introduced later as the project moves toward production readiness.

---

## 13. Terraform Workspaces vs Separate Directories

Terraform provides more than one way to manage environments.

One option is Terraform workspaces:

```bash
terraform workspace new dev
terraform workspace new prod
```

Then:

```bash
terraform workspace select dev
```

or:

```bash
terraform workspace select prod
```

Another approach is separate directories:

```text
environments/
├── dev/
├── test/
└── prod/
```

### Workspaces

Workspaces allow multiple state instances to use the same configuration.

They can be useful for certain use cases.

### Separate Directories

Separate directories make environment configuration more explicit.

For example:

```text
dev/
├── main.tf
└── terraform.tfvars

prod/
├── main.tf
└── terraform.tfvars
```

This makes it easier to see that production has a distinct configuration.

For this learning project, separate environment directories provide a clearer architecture and make the concepts easier to understand.

---

## 14. Avoiding Accidental Production Changes

Production infrastructure should be treated carefully.

Before applying production changes:

```bash
terraform plan
```

should always be reviewed.

A safer workflow is:

```text
Terraform Configuration
        ↓
terraform validate
        ↓
terraform plan
        ↓
Review
        ↓
Approval
        ↓
terraform apply
```

Production should not be changed blindly.

Later, CI/CD pipelines can enforce approval mechanisms for production deployments.

---

## 15. Environment Configuration and Modules

The relationship between environments and modules can be represented as:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.3; letter-spacing: 0px; font-size: 14px; background-color: #f8f9fa; padding: 15px; border-radius: 5px;">
                         Terraform
                            │
                    Reusable Modules
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
            DEV            TEST           PROD
             │              │              │
             │ inputs       │ inputs       │ inputs
             ▼              ▼              ▼
       AWS Resources   AWS Resources   AWS Resources
</pre>

The module defines **how** infrastructure is created.

The environment defines **how that infrastructure is configured for that environment**.

This separation is one of the most important Terraform architecture concepts.

---

## 16. Environment Structure for Employee Management API

A future environment-aware structure for the project can look like:

```text
terraform/
│
├── modules/
│   ├── dynamodb/
│   ├── iam/
│   └── lambda/
│
└── environments/
    │
    ├── dev/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars
    │
    ├── test/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars
    │
    └── prod/
        ├── main.tf
        ├── variables.tf
        └── terraform.tfvars
```

The development environment could use:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = var.table_name
  hash_key   = "employee_id"
}
```

with:

```hcl
table_name = "employee-dev-table"
```

Production could use the same module:

```hcl
module "dynamodb" {
  source = "../../modules/dynamodb"

  table_name = var.table_name
  hash_key   = "employee_id"
}
```

with:

```hcl
table_name = "employee-prod-table"
```

---

## 17. Environment Variables vs Module Variables

It is important to understand the distinction.

### Module Variable

Defined inside the reusable module:

```hcl
variable "table_name" {
  type = string
}
```

This describes what the module needs.

### Environment Value

Provided by the environment:

```hcl
table_name = "employee-dev-table"
```

This describes what the particular environment wants.

The flow is:

```text
Environment Value
       │
       ▼
Module Input Variable
       │
       ▼
Terraform Resource
```

Keeping this separation makes the infrastructure easier to maintain.

---

## 18. Environment Strategy for the Current Project

The Employee Management API is currently being developed as a learning project.

Therefore, the immediate goal is not to create a complicated enterprise environment structure.

The practical progression is:

```text
Current
   ↓
dev environment
   ↓
Reusable modules
   ↓
Environment-specific variables
   ↓
Separate state
   ↓
Remote backend
   ↓
test/staging
   ↓
production readiness
```

This prevents unnecessary complexity while still building the project toward a production-style architecture.

---

## 19. Learned vs Implemented

### 📚 Learned

The following concepts were learned:

* Terraform environments
* Environment isolation
* Environment-specific configuration
* Environment-specific variables
* `.tfvars` files
* Separate environment directories
* Resource naming conventions
* Environment-specific settings
* Separate Terraform state
* Terraform workspaces
* Separate directories vs workspaces
* Production protection
* Environment and module separation
* Environment-specific module inputs

### 🛠️ Implemented

The Employee Management API currently uses a development-focused infrastructure configuration.

The Terraform architecture separates reusable infrastructure modules from environment-specific configuration.

The intended structure is:

```text
terraform/
│
├── modules/
│   ├── dynamodb/
│   ├── iam/
│   └── lambda/
│
└── dev environment
```

Environment-specific values such as:

```text
employee-dev-table
employee-api-dev
```

are treated separately from the reusable module logic.

The project intentionally does not introduce unnecessary production environments at this stage.

### 🔮 Future Improvements

As the project becomes more production-oriented, the environment architecture can be expanded to:

```text
environments/
├── dev/
├── test/
└── prod/
```

Future improvements may include:

* separate Terraform state per environment
* remote backend
* environment-specific AWS accounts
* CI/CD deployment workflows
* production approval gates
* stronger environment isolation
* environment-specific IAM permissions
* environment-specific monitoring
* environment-specific security controls

These topics will be introduced progressively in later sprints.

---

## 20. Key Takeaways

> 1. Terraform environments allow infrastructure to be separated by deployment context.
> 2. Reusable modules should contain infrastructure logic rather than environment-specific values.
> 3. Environment configurations provide values to reusable modules.
> 4. `.tfvars` files can provide environment-specific configuration.
> 5. Separate directories can make environments explicit and easy to understand.
> 6. Each environment should ideally have isolated Terraform state.
> 7. Resource names should clearly identify their environment.
> 8. Different environments may require different infrastructure settings.
> 9. Terraform workspaces are another environment/state management option.
> 10. Separate directories provide clear configuration separation for this project.
> 11. Production infrastructure should always be reviewed through `terraform plan` before applying changes.
> 12. Environment management should evolve gradually rather than introducing unnecessary complexity too early.

Terraform environments complete the foundation built through Sprints 6–10. The next stage will move deeper into **Terraform Core concepts, state management, and remote backends**, which are essential for collaborative and production-grade Infrastructure as Code.
