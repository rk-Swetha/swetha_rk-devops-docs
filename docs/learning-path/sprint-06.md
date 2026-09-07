# Sprint 6 — Terraform Dependencies

## Overview

Sprint 6 focused on Terraform resource dependencies.

Terraform infrastructure contains multiple resources that depend on one another. For example, an AWS Lambda function may depend on an IAM role, while the Lambda function may interact with a DynamoDB table.

Understanding dependencies helps Terraform determine the correct order in which resources should be created, updated, and destroyed.

The Employee Management API project provided a practical example of these dependency relationships between IAM, Lambda, DynamoDB and API Gateway.

---

## 1. What are Terraform Dependencies?

A Terraform dependency exists when one resource requires another resource to exist or be configured first.

For example:

```text
IAM Role
    ↓
Lambda Function
```

The Lambda function requires an IAM role to execute.

Terraform uses dependencies to build an internal dependency graph and determine the correct execution order.

---

## 2. Implicit Dependencies

Terraform can automatically detect dependencies when one resource references another resource.

For example:

```hcl
resource "aws_iam_role" "lambda_role" {

  name = "employee-api-role"

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

resource "aws_lambda_function" "employee_api" {

  function_name = "employee-api"

  role = aws_iam_role.lambda_role.arn

  runtime = "python3.13"
  handler = "app.lambda_handler"

  filename = "lambda.zip"
}
```

The following reference creates an implicit dependency:

```hcl
role = aws_iam_role.lambda_role.arn
```

Terraform understands that:

```text
aws_lambda_function.employee_api
                ↓
depends on
                ↓
aws_iam_role.lambda_role
```

Therefore, Terraform creates the IAM role before the Lambda function.

---

## 3. Explicit Dependencies

Sometimes Terraform cannot automatically determine a dependency.

Terraform provides the `depends_on` argument for explicitly defining such dependencies.

For example:

```hcl
resource "aws_lambda_function" "employee_api" {

  function_name = "employee-api"

  role = aws_iam_role.lambda_role.arn

  runtime = "python3.13"
  handler = "app.lambda_handler"

  filename = "lambda.zip"

  depends_on = [
    aws_iam_role_policy_attachment.lambda_policy_attachment
  ]
}
```

This tells Terraform that the Lambda function must wait for the IAM policy attachment.

Conceptually:

```text
IAM Role
    ↓
IAM Policy
    ↓
Policy Attachment
    ↓
Lambda
```

---

## 4. Implicit vs Explicit Dependencies

Terraform supports both implicit and explicit dependencies.

### Implicit Dependency

Created automatically through resource references.

```hcl
role = aws_iam_role.lambda_role.arn
```

### Explicit Dependency

Defined manually using:

```hcl
depends_on = [...]
```

The recommended approach is to prefer implicit dependencies whenever possible.

```text
Implicit Dependency
        ↓
Preferred

Explicit depends_on
        ↓
Use only when required
```

---

## 5. Why Dependencies Matter

Correct dependency management helps Terraform:

* Create resources in the correct order.
* Update resources safely.
* Destroy resources in the correct order.
* Understand relationships between resources.
* Avoid unnecessary deployment failures.
* Build an accurate infrastructure dependency graph.

Without proper dependency relationships, Terraform may attempt operations before required resources are ready.

---

## 6. Dependency Relationships in the Project

The Employee Management API contains several infrastructure relationships.

A simplified dependency structure is:

```text
IAM Role
    │
    ├── IAM Policy
    │
    └── Policy Attachment
            │
            ↓
         Lambda
            │
            ├── DynamoDB
            │
            └── API Gateway
```

The important relationships include:

```text
IAM
 ↓
Lambda
```

Lambda requires an IAM execution role.

```text
DynamoDB
 ↓
Lambda
```

Lambda requires appropriate IAM permissions to access DynamoDB.

```text
Lambda
 ↓
API Gateway
```

API Gateway invokes the Lambda function.

---

## 7. Terraform Dependency Graph

Terraform can generate a dependency graph using:

```bash
terraform graph
```

The graph represents relationships between Terraform resources.

This is useful for understanding how Terraform determines the order of operations.

For the Employee Management API, the graph can be used to inspect relationships between:

* IAM
* Lambda
* DynamoDB
* API Gateway
* CloudWatch

---

## 8. When to Use `depends_on`

`depends_on` should not be added to every resource.

Terraform should normally determine dependencies automatically.

For example, this is preferable:

```hcl
role = aws_iam_role.lambda_role.arn
```

instead of unnecessarily adding:

```hcl
depends_on = [
  aws_iam_role.lambda_role
]
```

Explicit dependencies should be used when Terraform cannot determine a required dependency from the configuration itself.

---

## 9. Application to Employee Management API

The Employee Management API Terraform configuration was reviewed to understand how AWS resources depend on each other.

The main dependency relationships were:

```text
IAM Role
    ↓
Lambda

DynamoDB
    ↓
Lambda permissions

Lambda
    ↓
API Gateway
```

The project follows the principle of allowing Terraform to determine dependencies through resource references whenever possible.

---

## 10. Learned vs Implemented

### 📚 Learned

* Terraform dependencies
* Dependency graphs
* Implicit dependencies
* Explicit dependencies
* `depends_on`
* Resource references
* Resource creation order
* Resource destruction order

### 🛠️ Implemented

The Employee Management API infrastructure was reviewed for resource dependencies.

Important relationships between IAM, Lambda, DynamoDB and API Gateway were identified.

Terraform's implicit dependency mechanism was preferred wherever possible.

### 🔮 Future Improvements

Dependencies can be further improved by:

* Avoiding unnecessary `depends_on`.
* Reviewing dependency graphs during infrastructure changes.
* Keeping resource references explicit and clear.
* Adding explicit dependencies only when Terraform cannot infer them.
* Reviewing dependency relationships when introducing new modules.

---

## 11. Key Takeaways

> Terraform uses dependencies to determine the correct order for infrastructure operations.

> Resource references normally create implicit dependencies automatically.

> `depends_on` can be used when an explicit dependency is required.

> Implicit dependencies should generally be preferred over unnecessary explicit dependencies.

> Understanding dependencies becomes increasingly important as Terraform infrastructure grows.

Sprint 6 established the foundation for the next stage: **Terraform modules and reusable infrastructure**.
