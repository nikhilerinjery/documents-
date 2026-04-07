# 📚 Databricks Cluster Terraform Configuration - Complete Beginner's Guide

---

## 🎯 What Are We Building?

We are creating a **Databricks All-Purpose Cluster** using **Terraform** (Infrastructure as Code). This cluster will:
- Run data processing jobs (like importing data)
- Send logs and metrics to AWS CloudWatch for monitoring
- Be configurable for different environments (dev, QA, production)

---

## 📁 Files Overview

| File | Location | Purpose |
|------|----------|---------|
| **Root main.tf** | `dev-mumbai/.../main.tf` | Environment-specific configuration (the "caller") |
| **Module main.tf** | `modules/.../main.tf` | Reusable cluster template (the "module") |
| **variables.tf** | `modules/.../variables.tf` | Variable definitions with defaults |
| **compute_policy.tf** | `modules/.../compute_policy.tf` | Rules that constrain cluster settings |
| **cloudwatch_init.sh.tpl** | `modules/.../scripts/` | Script that installs CloudWatch agent |

---

## 📄 File 1: Root main.tf (Caller)

**Location:** `dev-mumbai/databricks/workspace-qa/redblack-cloud-common-cluster/main.tf`

This is your **environment-specific configuration**. Think of it as filling out a form to order a cluster.

### Section 1: Terraform Providers

```terraform
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
|------|-------------|
| `terraform` block | Tells Terraform what external services we need |
| `databricks` provider | Allows Terraform to create Databricks resources |
| `aws` provider | Allows Terraform to access AWS services (like SSM) |
| `version = "~> 5.0"` | Use AWS provider version 5.x |

---

### Section 2: AWS SSM Parameter (Secrets)

```terraform
data "aws_ssm_parameter" "databricks_credentials_data" {
  name              = "rb-dev-mumbai-databricks-account-secrets"
  with_decryption   = true
}
```

| Item | Explanation |
|------|-------------|
| `data` block | Reads existing data (doesn't create anything) |
| `aws_ssm_parameter` | AWS Systems Manager Parameter Store - secure secret storage |
| `name` | The parameter name storing Databricks credentials |
| `with_decryption = true` | Decrypt the secret (it's encrypted at rest) |

**Why?** Databricks needs credentials (client_id, client_secret) to authenticate. We store them securely in AWS SSM, not in code.

---

### Section 3: Local Variables

```terraform
locals {
  databricks_credentials = jsondecode(data.aws_ssm_parameter.databricks_credentials_data.value)
  scriptPath             = "Terraform/dev-mumbai/databricks/workspace-qa/redblack-cloud-common-cluster"
  instance_profile_arn   = "arn:aws:iam::621010003962:instance-profile/rb-dev-mumbai-databricks-worker-common-instance-profile"
  environment            = "dev-mumbai"
}
```

| Variable | Value | Explanation |
|----------|-------|-------------|
| `databricks_credentials` | JSON object | Parses the SSM secret into usable format |
| `scriptPath` | Folder path | Used for tagging - shows where this Terraform code lives |
| `instance_profile_arn` | AWS IAM ARN | Gives cluster permission to access AWS services (S3, CloudWatch) |
| `environment` | `"dev-mumbai"` | Environment name for tagging and logging |

---

### Section 4: Databricks Provider Configuration

```terraform
provider "databricks" {
  alias         = "workspaces"
  host          = "https://dbc-cea3802f-f426.cloud.databricks.com/"
  client_id     = local.databricks_credentials["client_id"]
  client_secret = local.databricks_credentials["client_secret"]
}
```

| Item | Explanation |
|------|-------------|
| `alias = "workspaces"` | Name this provider configuration (allows multiple providers) |
| `host` | Databricks workspace URL (your Databricks account) |
| `client_id` | Service principal ID from SSM secrets |
| `client_secret` | Service principal secret from SSM secrets |

---

### Section 5: Module Call (The Main Configuration)

```terraform
module "redblack_common_cluster" {
  source    = "../../../../modules/databricks/all_purpose_common_cluster"
  providers = {databricks = databricks.workspaces}
  
  # All parameters below...
}
```

#### Basic Cluster Settings:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `script_path` | `local.scriptPath` | Tag showing Terraform code location |
| `environment` | `"dev-mumbai"` | Environment name (dev/qa/prod) |
| `single_user_name` | `"20f0e591-bf3a-42bb-9cdd-3fdabbb8766e"` | **User ID** who owns this cluster (SINGLE_USER mode means only this user can run jobs) |
| `cluster_name` | `"redblack-cloud-common-cluster"` | Display name of your cluster in Databricks UI |
| `spark_version` | `"15.4.x-scala2.12"` | Databricks Runtime version (includes Spark + libraries) |

#### Node Configuration:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `node_type_id` | `"m7g.large"` | **Worker node** EC2 instance type (2 vCPU, 8GB RAM, ARM-based) |
| `driver_node_type_id` | `"m7g.large"` | **Driver node** EC2 instance type (coordinates the work) |
| `min_workers` | `1` | Minimum number of worker nodes |
| `max_workers` | `1` | Maximum number of worker nodes |

**Visual:**
```
┌─────────────────┐     ┌─────────────────┐
│  Driver Node    │────▶│  Worker Node    │
│  (m7g.large)    │     │  (m7g.large)    │
│  Coordinates    │     │  Does the work  │
└─────────────────┘     └─────────────────┘
```

#### Autotermination:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `autotermination_minutes` | `30` | Cluster shuts down after 30 minutes of inactivity (saves money!) |

#### Tags:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `custom_tags` | `{}` | Extra tags you can add (empty here) |

#### Instance Profile:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `instance_profile_arn` | `"arn:aws:iam::...instance-profile..."` | **IAM Role** that cluster nodes assume to access AWS (S3, CloudWatch) |

#### Spot Instance Settings:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `availability` | `"SPOT_WITH_FALLBACK"` | Use cheap Spot instances, fall back to On-Demand if unavailable |

**Spot Instance Options:**

| Option | Cost | Reliability | Best For |
|--------|------|-------------|----------|
| `SPOT` | Cheapest (70-90% off) | Can be interrupted | Non-critical batch jobs |
| `SPOT_WITH_FALLBACK` | Cheap | Better - uses On-Demand if no Spot | Most workloads ✅ |
| `ON_DEMAND` | Full price | Most reliable | Critical production |

#### Policy Constraint Settings:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `autotermination_max_minutes` | `60` | **Policy rule:** Users can't set autotermination higher than 60 mins |
| `autotermination_minutes_min_value` | `30` | **Policy rule:** Users must set autotermination at least 30 mins |
| `first_on_demand_min_value` | `1` | **Policy rule:** Minimum On-Demand nodes before using Spot |
| `first_on_demand_max_value` | `10` | **Policy rule:** Maximum On-Demand nodes before using Spot |
| `ebs_volume_count` | `1` | Number of extra EBS storage volumes per node |
| `autoscale_max_workers_max_value` | `10` | **Policy rule:** Maximum workers can't exceed 10 |
| `autoscale_min_workers_max_value` | `10` | **Policy rule:** Min workers setting can't exceed 10 |

**What is `first_on_demand`?**
```
first_on_demand = 1 means:
┌──────────────────────────────────────────┐
│ Node 1: ON-DEMAND (always reliable)      │
│ Node 2: SPOT (cheap, can be interrupted) │
│ Node 3: SPOT                             │
│ Node 4: SPOT                             │
└──────────────────────────────────────────┘
```

#### CloudWatch Logging Settings:

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `region` | `"ap-south-1"` | AWS region for CloudWatch (Mumbai) |
| `env_id` | `"dev-mumbai"` | Used in log group path: `/rb/databricks/import2-0/dev-mumbai/...` |
| `app_name` | `"rbcloud"` | Application name for CloudWatch metrics |
| `job_name` | `"import2-0"` | Job name in log path: `/rb/databricks/import2-0/...` |
| `enable_monitoring_and_spark_logs` | `true` | Enable/disable CloudWatch logging |

---

### Section 6: Permissions

```terraform
resource "databricks_permissions" "redblack_common_cluster_access_controls" {
  provider   = databricks.workspaces
  cluster_id = module.redblack_common_cluster.cluster_id

  dynamic "access_control" {
    for_each = [
      { group_name = "developers", permission_level = "CAN_ATTACH_TO" },
      { group_name = "devops_team", permission_level = "CAN_MANAGE" },
    ]
    content {
      group_name       = access_control.value.group_name
      permission_level = access_control.value.permission_level
    }
  }
}
```

| Group | Permission | Explanation |
|-------|------------|-------------|
| `developers` | `CAN_ATTACH_TO` | Can run notebooks/jobs on this cluster |
| `devops_team` | `CAN_MANAGE` | Full control (start, stop, configure, delete) |

---

## 📄 File 2: Module main.tf

**Location:** `modules/databricks/all_purpose_common_cluster/main.tf`

This is the **reusable template** that creates the actual cluster.

### Section 1: CloudWatch Script Template

```terraform
locals {
  script_content = templatefile("${path.module}/scripts/cloudwatch_init.sh.tpl", {
    REGION           = var.region
    ENV_ID           = var.env_id
    APP_NAME         = var.app_name
    JOB_NAME         = var.job_name
    DB_CLUSTER_NAME  = var.cluster_name
  })
}
```

| Item | Explanation |
|------|-------------|
| `templatefile()` | Reads a template file and replaces variables |
| `REGION` | AWS region for CloudWatch agent |
| `ENV_ID` | Environment name for log paths |
| `APP_NAME` | Application name for metrics |
| `JOB_NAME` | Job name for log paths |
| `DB_CLUSTER_NAME` | Cluster name for log stream names |

---

### Section 2: Workspace File (Upload Script)

```terraform
resource "databricks_workspace_file" "cloudwatch_init" {
  count          = var.enable_monitoring_and_spark_logs ? 1 : 0
  path           = "/Shared/scripts/cloudwatch_init.sh"
  content_base64 = base64encode(local.script_content)
}
```

| Item | Explanation |
|------|-------------|
| `count = ... ? 1 : 0` | Create file only if monitoring enabled (conditional) |
| `path` | Where to store the script in Databricks workspace |
| `content_base64` | The script content, encoded as base64 |

---

### Section 3: Cluster Resource

```terraform
resource "databricks_cluster" "common_cluster" {
  data_security_mode            = "SINGLE_USER"
  num_workers                   = var.min_workers
  is_single_node                = "false"
  cluster_name                  = var.cluster_name
  spark_version                 = var.spark_version
  autotermination_minutes       = var.autotermination_minutes
  node_type_id                  = var.node_type_id
  driver_node_type_id           = var.driver_node_type_id
  enable_local_disk_encryption  = true
  single_user_name              = var.single_user_name
  no_wait                       = true
  ...
}
```

| Attribute | Value | Explanation |
|-----------|-------|-------------|
| `data_security_mode` | `"SINGLE_USER"` | Only one user can access - provides data isolation |
| `num_workers` | `var.min_workers` | Initial number of workers |
| `is_single_node` | `"false"` | Multi-node cluster (driver + workers) |
| `cluster_name` | `var.cluster_name` | Display name |
| `spark_version` | `var.spark_version` | Databricks Runtime version |
| `autotermination_minutes` | `var.autotermination_minutes` | Auto-shutdown time |
| `enable_local_disk_encryption` | `true` | Encrypt local SSDs (security!) |
| `single_user_name` | `var.single_user_name` | User who owns the cluster |
| `no_wait` | `true` | Don't wait for cluster to start (faster terraform apply) |

---

### Section 4: Autoscale Block

```terraform
autoscale {
  min_workers = var.min_workers
  max_workers = var.max_workers
}
```

| Attribute | Explanation |
|-----------|-------------|
| `min_workers` | Minimum workers always running |
| `max_workers` | Maximum workers during high load |

**Visual - Autoscaling:**
```
Low Load:     1 worker  (min_workers = 1)
              ┌───┐
              │ W │
              └───┘

High Load:    5 workers (scales up to max_workers)
              ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
              │ W │ │ W │ │ W │ │ W │ │ W │
              └───┘ └───┘ └───┘ └───┘ └───┘
```

---

### Section 5: AWS Attributes

```terraform
aws_attributes {
  availability            = var.availability
  ebs_volume_count        = var.ebs_volume_count
  ebs_volume_size         = 32
  zone_id                 = "auto"
  first_on_demand         = 1
  spot_bid_price_percent  = 100
  ebs_volume_type         = "GENERAL_PURPOSE_SSD"
  instance_profile_arn    = var.instance_profile_arn
}
```

| Attribute | Value | Explanation |
|-----------|-------|-------------|
| `availability` | `var.availability` | SPOT_WITH_FALLBACK, ON_DEMAND, or SPOT |
| `ebs_volume_count` | `var.ebs_volume_count` | Extra storage volumes per node |
| `ebs_volume_size` | `32` | Size of each extra volume (32 GB) |
| `zone_id` | `"auto"` | Let AWS choose best availability zone |
| `first_on_demand` | `1` | First N nodes are On-Demand (reliable) |
| `spot_bid_price_percent` | `100` | Pay up to 100% of On-Demand price for Spot |
| `ebs_volume_type` | `"GENERAL_PURPOSE_SSD"` | SSD storage type (gp2/gp3) |
| `instance_profile_arn` | `var.instance_profile_arn` | IAM role for AWS access |

---

### Section 6: Custom Tags

```terraform
custom_tags = merge({
  Environment   = var.environment,
  Application   = "rbcloud",
  CreatedUsing  = "Terraform",
  ScriptPath    = var.script_path,
  type_tag      = "databricks_worker",
}, var.custom_tags)
```

| Tag | Value | Explanation |
|-----|-------|-------------|
| `Environment` | `"dev-mumbai"` | Which environment |
| `Application` | `"rbcloud"` | Application name |
| `CreatedUsing` | `"Terraform"` | Shows this was created by Terraform |
| `ScriptPath` | Folder path | Where the Terraform code lives |
| `type_tag` | `"databricks_worker"` | Type of resource |

**Why tags?** For billing, organization, and finding resources in AWS console.

---

### Section 7: Init Scripts (CloudWatch Setup)

```terraform
dynamic "init_scripts" {
  for_each = var.enable_monitoring_and_spark_logs ? [1] : []
  content {
    workspace {
      destination = databricks_workspace_file.cloudwatch_init[0].path
    }
  }
}
```

| Item | Explanation |
|------|-------------|
| `dynamic` | Conditionally create this block |
| `for_each = ... ? [1] : []` | If enabled, create 1 block; if disabled, create 0 |
| `destination` | Path to the init script in workspace |

**What is an init script?** A shell script that runs when each node starts. Here it installs and configures CloudWatch agent.

---

## 📄 File 3: cloudwatch_init.sh.tpl

**Location:** `modules/.../scripts/cloudwatch_init.sh.tpl`

This script runs on **every node** when the cluster starts.

### What it does:
1. ✅ Downloads and installs AWS CloudWatch agent
2. ✅ Detects if this is Driver or Worker node
3. ✅ Configures log collection (Spark logs, stderr, stdout)
4. ✅ Configures metrics (disk usage, memory usage)
5. ✅ Starts the CloudWatch agent

### CloudWatch Log Groups Created:
```
/rb/databricks/import2-0/dev-mumbai/driver/spark-log   ← Spark application logs
/rb/databricks/import2-0/dev-mumbai/driver/stderr      ← Error output
/rb/databricks/import2-0/dev-mumbai/driver/stdout      ← Standard output
/rb/databricks/import2-0/dev-mumbai/executor/stderr    ← Worker errors
/rb/databricks/import2-0/dev-mumbai/executor/stdout    ← Worker output
```

### CloudWatch Metrics Collected:

| Metric | Explanation |
|--------|-------------|
| `dataset_driver_disk_used_percent` | How full is the driver's disk? |
| `dataset_driver_mem_used_percent` | How much RAM is the driver using? |
| `dataset_executor_disk_used_percent` | How full is the worker's disk? |
| `dataset_executor_mem_used_percent` | How much RAM is the worker using? |

---

## 🔄 Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TERRAFORM APPLY                                  │
└─────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  1. ROOT main.tf (dev-mumbai)                                           │
│     - Reads credentials from AWS SSM                                    │
│     - Calls the module with environment-specific values                 │
└─────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  2. MODULE (all_purpose_common_cluster)                                 │
│     - Creates workspace file (init script)                              │
│     - Creates cluster with all settings                                 │
│     - Applies compute policy                                            │
└─────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  3. DATABRICKS CLUSTER STARTS                                           │
│     ┌─────────────┐                                                     │
│     │   Driver    │ ── init script runs ── CloudWatch agent installed   │
│     └─────────────┘                                                     │
│     ┌─────────────┐                                                     │
│     │   Worker    │ ── init script runs ── CloudWatch agent installed   │
│     └─────────────┘                                                     │
└─────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  4. CLOUDWATCH RECEIVES DATA                                            │
│     - Logs: /rb/databricks/import2-0/dev-mumbai/...                    │
│     - Metrics: Databricks/AllPurpose namespace                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table: All Variables

| Variable | Example Value | Category | Purpose |
|----------|---------------|----------|---------|
| `cluster_name` | `"redblack-cloud-common-cluster"` | Basic | Cluster display name |
| `spark_version` | `"15.4.x-scala2.12"` | Basic | Databricks Runtime version |
| `node_type_id` | `"m7g.large"` | Compute | Worker EC2 instance type |
| `driver_node_type_id` | `"m7g.large"` | Compute | Driver EC2 instance type |
| `min_workers` | `1` | Scaling | Minimum workers |
| `max_workers` | `1` | Scaling | Maximum workers |
| `autotermination_minutes` | `30` | Cost | Auto-shutdown after idle |
| `single_user_name` | `"20f0e591-..."` | Security | Owner user ID |
| `environment` | `"dev-mumbai"` | Tagging | Environment name |
| `script_path` | `"Terraform/..."` | Tagging | Code location |
| `instance_profile_arn` | `"arn:aws:iam::..."` | Security | AWS IAM role for nodes |
| `availability` | `"SPOT_WITH_FALLBACK"` | Cost | Spot vs On-Demand |
| `ebs_volume_count` | `1` | Storage | Extra EBS volumes |
| `first_on_demand_min_value` | `1` | Policy | Min On-Demand nodes |
| `first_on_demand_max_value` | `10` | Policy | Max On-Demand nodes |
| `autotermination_max_minutes` | `60` | Policy | Max idle time allowed |
| `autotermination_minutes_min_value` | `30` | Policy | Min idle time required |
| `autoscale_max_workers_max_value` | `10` | Policy | Max workers limit |
| `autoscale_min_workers_max_value` | `10` | Policy | Min workers limit |
| `region` | `"ap-south-1"` | Logging | AWS region for CloudWatch |
| `env_id` | `"dev-mumbai"` | Logging | Environment in log paths |
| `app_name` | `"rbcloud"` | Logging | App name in metrics |
| `job_name` | `"import2-0"` | Logging | Job name in log paths |
| `enable_monitoring_and_spark_logs` | `true` | Logging | Enable/disable CloudWatch |

---

## ❓ Common Questions

**Q: What's the difference between `min_workers` and `autoscale_min_workers_max_value`?**
- `min_workers = 1` → **This cluster** has 1 worker minimum
- `autoscale_min_workers_max_value = 10` → **Policy rule** that prevents users from setting min_workers above 10

**Q: Why use Spot instances?**
- 70-90% cheaper than On-Demand
- `SPOT_WITH_FALLBACK` uses Spot when available, On-Demand otherwise
- Good for fault-tolerant batch jobs

**Q: What is `single_user_name`?**
- A user ID (Azure AD / Databricks) that owns the cluster
- Only this user can run code on it (security feature)

**Q: Why so many "min_value" and "max_value" variables?**
- These are **cluster policy constraints**
- They limit what users can configure when using this cluster
- Prevents overspending or misconfiguration

---

*Document generated on April 7, 2026*
*Databricks All-Purpose Cluster Terraform Module Documentation*
