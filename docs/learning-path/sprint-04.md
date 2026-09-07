# Sprint 4 — Terraform Variables

## Overview

Sprint 4 focused on Terraform input variables and the importance of separating configurable values from infrastructure logic.

The main goal was to reduce hardcoded values and make the Terraform configuration reusable across different environments.

---

## 1. Why Variables?

Hardcoding environment-specific values directly inside resources makes infrastructure harder to reuse.

For example:

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

The configuration is tightly coupled to the development environment.

If the same infrastructure needs to be created for QA or production, the configuration would need to be modified.

Terraform variables provide a cleaner approach.

---

## 2. Input Variables

An input variable allows values to be supplied to Terraform configuration.

Example:

```hcl
variable "table_name" {
  type        = string
  description = "Name of the DynamoDB table"
}
```

The variable can then be referenced using:

```hcl
var.table_name
```

---

## 3. Using Variables in Resources

Instead of:

```hcl
name = "employee-dev-table"
```

the resource can use:

```hcl
name = var.table_name
```

Example:

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

Now the resource configuration does not need to know the environment-specific table name.

---

## 4. Variable Types

Terraform supports different variable types.

Common types include:

```text
string
number
bool
list
set
map
object
tuple
```

Example:

```hcl
variable "environment" {
  type = string
}
```

Another example:

```hcl
variable "memory_size" {
  type = number
}
```

Using types helps Terraform understand what kind of value a variable should receive.

---

## 5. `terraform.tfvars`

Variable values can be supplied through a `.tfvars` file.

Example:

```hcl
table_name  = "employee-dev-table"
environment = "dev"
```

Terraform automatically loads values from a file named:

```text
terraform.tfvars
```

Other variable files can also be supplied explicitly when required.

---

## 6. Separating Configuration from Infrastructure Logic

One of the most important ideas from this sprint was:

> **Infrastructure logic should be reusable, while environment-specific values should be configurable.**

For example:

### Infrastructure logic

```hcl
resource "aws_dynamodb_table" "employee" {
  name         = var.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "employee_id"
}
```

### Environment-specific configuration

```hcl
table_name = "employee-dev-table"
```

The resource definition does not need to change when the environment changes.

---

## 7. Before and After

### Before variables

```hcl
resource "aws_dynamodb_table" "employee" {
  name = "employee-dev-table"
}
```

The environment is hardcoded.

### After variables

```hcl
resource "aws_dynamodb_table" "employee" {
  name = var.table_name
}
```

Value:

```hcl
table_name = "employee-dev-table"
```

This provides a cleaner separation between infrastructure logic and configuration.

---

## 8. Variables and Environment Reusability

Variables became important when the project moved toward multiple environments.

Conceptually:

<pre style="font-family: 'Courier New', Courier, monospace; line-height: 1.2; letter-spacing: 0px;">

                 Shared Terraform Logic
                            │
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
           Dev             QA             Prod
            │               │               │
            ↓               ↓               ↓
       dev values       QA values      prod values
</pre>

The infrastructure logic can remain reusable while values differ by environment.

---

## 9. Variables in Modules

Variables later became an important part of reusable Terraform modules.

For example, a Lambda module can accept:

```hcl
variable "function_name" {
  type = string
}

variable "runtime" {
  type = string
}

variable "memory_size" {
  type = number
}
```

The module can then use:

```hcl
function_name = var.function_name
runtime       = var.runtime
memory_size   = var.memory_size
```

This allows the same module to be reused for different environments.

---

## 10. Application to Employee Management API

Variables were introduced to remove hardcoded infrastructure values from the Employee Management API Terraform configuration.

For example, the DynamoDB table name evolved from a hardcoded value:

```hcl
name = "employee-dev-table"
```

to a variable-driven configuration:

```hcl
name = var.table_name
```

This approach later supported the project's reusable module and environment architecture.

---

## 11. Learned vs Implemented

### 📚 Learned

* Input variables
* Variable declarations
* Variable types
* Variable values
* `terraform.tfvars`
* Variable references
* Separating configuration from infrastructure logic
* Environment reusability

### 🛠️ Implemented

Terraform variables were used to replace hardcoded infrastructure values in the Employee Management API project.

The DynamoDB table name was one example of configuration being moved into a variable.

Variables later became part of the reusable module architecture.

### 🔮 Future Improvements

Variables can be further strengthened with:

* Validation rules
* Sensitive variables where appropriate
* Structured object variables
* Environment-specific variable files
* More comprehensive input validation

---

## 12. Key Takeaways

> Variables allow Terraform configurations to accept configurable input values.

> Variables reduce unnecessary hardcoding.

> Variable types make configuration more predictable.

> `terraform.tfvars` can provide values for Terraform input variables.

> Variables are essential for creating reusable modules and environment-specific infrastructure.

The introduction of variables was an important step toward the reusable Terraform architecture developed in later sprints.
