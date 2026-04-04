#  Databricks Cluster Terraform Configuration

We are creating a Databricks All-Purpose Cluster using Terraform (Infrastructure as Code). This cluster will:

1. Run data processing jobs (like importing data)
2. Send logs and metrics to AWS CloudWatch for monitoring
3. Be configurable for different environments (dev, QA, production)


## Root main.tf (Caller)
```
terraform {
  required_providers {
    databricks = {
      source = "databricks/databricks"
    }
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```
| Item | Explanation |
| :--- | :--- |
| **terraform block** | Tells Terraform what external services we need |
| **databricks provider** | Allows Terraform to create Databricks resources |
| **aws provider** | Allows Terraform to access AWS services (like SSM) |
| **version = "~> 5.0"** | Use AWS provider version 5.x |
