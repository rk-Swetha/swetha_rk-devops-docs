# Sprint 6 — Terraform Dependencies

## Overview

Terraform does not simply create resources in the order they appear in a `.tf` file.

Instead, Terraform builds a **dependency graph** to determine which resources must be created, updated, or destroyed before other resources.

Understanding dependencies is important because real-world infrastructure rarely consists of completely independent resources.

For example, in the Employee Management API project:

```text
DynamoDB Table
      ↑
      │
Lambda IAM Policy
      ↑
      │
Lambda IAM Role
      ↑
      │
Lambda Function
```

The Lambda function depends on the IAM role, and the IAM role may depend on policies that grant access to DynamoDB.

Terraform uses these relationships to determine the correct execution order.

---

## 1. What is a Terraform Dependency?

A dependency means that one Terraform resource relies on another resource.

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

A Lambda function may need permission to access this DynamoDB table.

Therefore, the infrastructure has a logical relationship:

```text
DynamoDB
   │
   │ access permission
   ▼
IAM Policy
   │
   ▼
IAM Role
   │
   ▼
Lambda Function
```

Terraform needs to understand these relationships so that resources are provisioned safely.

---

## 2. Implicit Dependencies

An **implicit dependency** occurs when Terraform can determine the relationship automatically from resource references.

Example:

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

Suppose an IAM policy references the DynamoDB table ARN:

```hcl
resource "aws_iam_policy" "lambda_dynamodb" {
  name = "lambda-dynamodb-policy"

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "dynamodb:GetItem",
        "dynamodb:PutItem"
      ]

      Resource = aws_dynamodb_table.employee.arn
    }]
  })
}
```

Terraform sees:

```text
aws_iam_policy.lambda_dynamodb
              │
              │ references
              ▼
aws_dynamodb_table.employee
```

Terraform therefore knows that the DynamoDB table must exist before the policy can correctly reference its ARN.

This is an **implicit dependency**.

---

## 3. Explicit Dependencies

Sometimes Terraform cannot automatically determine a dependency.

In such cases, Terraform provides the `depends_on` meta-argument.

Example:

```hcl
resource "aws_lambda_function" "employee_api" {
  function_name = "employee-api-dev"

  # other configuration...

  depends_on = [
    aws_iam_role_policy_attachment.lambda_policy
  ]
}
```

This explicitly tells Terraform:

```text
Create IAM policy attachment
          ↓
Then create/update Lambda
```

---

## 4. `depends_on` Syntax

The general syntax is:

```hcl
depends_on = [
  resource.type.name
]
```

For example:

```hcl
depends_on = [
  aws_iam_role_policy_attachment.lambda_dynamodb
]
```

Multiple dependencies can also be specified:

```hcl
depends_on = [
  aws_iam_role_policy_attachment.lambda_dynamodb,
  aws_iam_role_policy_attachment.lambda_logs
]
```

This means Terraform must consider all listed resources before proceeding.

---

## 5. Implicit vs Explicit Dependencies

The two approaches can be compared as follows:

| Dependency | How Terraform knows | Example                           |
| ---------- | ------------------- | --------------------------------- |
| Implicit   | Resource reference  | `aws_dynamodb_table.employee.arn` |
| Explicit   | `depends_on`        | `depends_on = [...]`              |

### Prefer implicit dependencies

Whenever possible, use resource references.

For example:

```hcl
Resource = aws_dynamodb_table.employee.arn
```

is generally preferable to:

```hcl
depends_on = [
  aws_dynamodb_table.employee
]
```

when the reference itself already establishes the dependency.

---

## 6. Why `depends_on` Should Not Be Overused

It is tempting to add `depends_on` everywhere.

This is usually unnecessary.

For example:

```hcl
resource "aws_iam_policy" "lambda_dynamodb" {
  # ...
  
  depends_on = [
    aws_dynamodb_table.employee
  ]
}
```

If the policy already contains:

```hcl
Resource = aws_dynamodb_table.employee.arn
```

then Terraform already knows about the dependency.

Adding `depends_on` provides no additional value.

Overusing explicit dependencies can make Terraform configurations:

* harder to understand
* more tightly coupled
* less flexible
* harder to maintain

The preferred approach is:

```text
Resource Reference
       ↓
Implicit Dependency
       ↓
Use depends_on only when necessary
```

---

## 7. Dependency Graph

Terraform internally creates a dependency graph.

For the Employee Management API, a simplified graph could look like:

```text
                    ┌─────────────────┐
                    │    DynamoDB     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   IAM Policy    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     IAM Role    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Lambda Function  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  API Gateway    │
                    └─────────────────┘
```

Terraform uses this graph to determine the correct execution order.

---

## 8. Terraform Graph Command

Terraform provides a command to visualize the dependency graph:

```bash
terraform graph
```

Example:

```bash
terraform graph
```

The output is generally represented in Graphviz DOT format.

It can be redirected to a file:

```bash
terraform graph > graph.dot
```

The graph can then be rendered using Graphviz or another compatible visualization tool.

This is particularly useful when troubleshooting complicated Terraform configurations.

---

## 9. Dependency and Resource Destruction

Dependencies also affect resource destruction.

Suppose:

```text
Lambda
  ↓
IAM Role
  ↓
IAM Policy
```

Terraform considers these relationships when determining the appropriate destruction order.

The goal is to avoid attempting operations against resources before their dependencies have been handled.

This is one reason Terraform's dependency graph is fundamental to infrastructure lifecycle management.

---

## 10. Dependency Example in the Project

A simplified project configuration may contain:

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

IAM policy:

```hcl
resource "aws_iam_policy" "lambda_dynamodb" {
  name = "lambda-dynamodb-policy"

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "dynamodb:GetItem",
        "dynamodb:PutItem"
      ]

      Resource = aws_dynamodb_table.employee.arn
    }]
  })
}
```

The dependency is automatically established:

```text
DynamoDB Table
      │
      │ ARN reference
      ▼
IAM Policy
```

If a Lambda role policy attachment is then created:

```hcl
resource "aws_iam_role_policy_attachment" "lambda_dynamodb" {
  role       = aws_iam_role.lambda.name
  policy_arn = aws_iam_policy.lambda_dynamodb.arn
}
```

Terraform can build the chain:

```text
DynamoDB
   ↓
IAM Policy
   ↓
Policy Attachment
   ↓
Lambda Role / Function
```

---

## 11. Learned vs Implemented

### 📚 Learned

The following concepts were learned:

* Terraform dependency management
* Implicit dependencies
* Explicit dependencies
* `depends_on`
* Terraform dependency graphs
* Resource references
* Dependency-aware creation and destruction
* `terraform graph`
* Why unnecessary `depends_on` should be avoided

### 🛠️ Implemented

In the Employee Management API infrastructure, dependencies are primarily established through **resource references**.

For example:

```hcl
aws_dynamodb_table.employee.arn
```

creates an implicit relationship between the DynamoDB table and the IAM policy that references it.

The project follows the preferred approach of allowing Terraform to automatically determine dependencies wherever possible.

Explicit `depends_on` should only be introduced when Terraform cannot correctly infer the required relationship.

### 🔮 Future Improvements

As the infrastructure becomes more complex, dependency graphs can be reviewed using:

```bash
terraform graph
```

This can help identify:

* unexpected dependencies
* unnecessary dependencies
* tightly coupled resources
* potential infrastructure design problems

---

## 12. Key Takeaways

> 1. Terraform creates a dependency graph before applying infrastructure changes.

> 2. Resource references automatically create **implicit dependencies**.

> 3. `depends_on` creates an **explicit dependency**.

> 4. Implicit dependencies should generally be preferred.

> 5. `depends_on` should only be used when Terraform cannot infer the relationship.

> 6. Dependencies affect both resource creation and destruction.

> 7. `terraform graph` can help visualize complex infrastructure relationships.

The next step is to move from individual Terraform resources toward **reusable infrastructure components using Terraform Modules**.
