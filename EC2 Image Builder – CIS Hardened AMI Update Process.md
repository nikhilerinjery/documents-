# EC2 Image Builder – CIS Hardened AMI Update Process

The work consists of four independent update flows:

1. ECS Custom Image (CIS L1 – EKS Optimized Amazon Linux 2)
2. Jenkins Linux Agent Image (CIS L1 – EKS Optimized Amazon Linux 2)
3. Amazon Linux FSx Cleanup Agent Image (CIS L1 – Amazon Linux 2 Kernel 5.10)
4. Troubleshoot EC2 Instance Image (CIS L1 – EKS Optimized Amazon Linux 2)


---

## Step 1: ECS Custom Image Update

### Source AMI:
**CIS Hardened Image Level 1 – EKS-Optimized Amazon Linux 2**

---

### 1. Retrieve Latest AMI From AWS Marketplace

- Go to **AWS Marketplace → Manage Subscriptions**
- Open **CIS Hardened Image Level 1 on EKS‑Optimized Amazon Linux 2**
- Note the **latest stable AMI ID**

---

### 2. Update Image Builder Pipeline

Update the AMI inside:

infra-as-code/Terraform/staging/ec2-image-builder/amazon-linux-ecs-custom-image-cis-1

Run Image Builder pipeline to generate a new custom AMI.

---

### 3. Update All Dependent Files With New Custom AMI

#### CloudFormation Files to Update
```
CloudFormation/Shared/Linux_ECS_Cluser_parameters.json
CloudFormation/Staging/ECSCluster/Linux_2_ECS_Cluser_parameters.json
CloudFormation/Staging/ECSCluster/Linux_ECS_Cluser_parameters.json
CloudFormation/UAT/ECSCluster/Linux_2_ECS_Cluser_parameters.json
CloudFormation/UAT/ECSCluster/Linux_ECS_Cluser_parameters.json
```
#### Terraform Files to Update
```
Terraform/staging/ec2-image-builder/amazon-linux-ecs-custom-image-cis-1/main.tf
Terraform/staging/ec2-image-builder/jenkins-agent-linux-custom-image/main.tf
Terraform/staging/ec2-image-builder/troubleshoot-linux-custom-image/main.tf
```
---

---

## Step 2: Jenkins Linux Agent Image Update

### Source AMI:
**CIS Hardened Image Level 1 – EKS‑Optimized Amazon Linux 2**

---

### 1. Get Latest AMI From AWS Marketplace
Same process as Step 1.

---

### 2. Update the Image Builder Pipeline

Update AMI reference in:


infra-as-code/Terraform/staging/ec2-image-builder/jenkins-agent-linux-custom-image

Trigger Image Builder to produce a new Jenkins Agent AMI.

---

### 3. Update Terraform References

```
Terraform/shared/jenkins-linux-agent/main.tf
Terraform/shared/jenkins-linux-agent-secondary/main.tf
Terraform/staging/jenkins-linux-agent/main.tf
Terraform/staging/jenkins-linux-agent-secondary/main.tf
Terraform/uat/jenkins-linux-agent/main.tf
Terraform/uat/jenkins-linux-agent-secondary/main.tf
```
---

---

## Step 3: FSx Cleanup Spot Agent Image Update

### Source AMI:
**CIS Hardened Image Level 1 – Amazon Linux 2 Kernel 5.10**

---

### 1. Get Latest AMI

Go to AWS Marketplace → Get latest AMI ID for  
**CIS Hardened Image Level 1 on Amazon Linux 2 Kernel 5.10**

---

### 2. Update Image Builder Pipeline

Modify AMI reference in:


infra-as-code/Terraform/staging/ec2-image-builder/amazon-linux-fsx-custom-image-cis

Trigger build → generates new FSx Cleanup image.

---

### 3. Update Terraform Files

```
Terraform/staging/fsx-cleanup-spot-agent/main.tf
Terraform/uat/fsx-cleanup-spot-agent/main.tf
```
---

---

## Step 4: Troubleshoot EC2 Image Update

### Source AMI:
**CIS Hardened Image Level 1 – EKS‑Optimized Amazon Linux 2**

---

### 1. Get Latest AMI
Same Marketplace source as Step 1.

---

### 2. Update Pipeline


infra-as-code/Terraform/staging/ec2-image-builder/amazon-linux-ecs-custom-image-cis-1

Run Image Builder → get new AMI ID.

---

### 3. Update Terraform Files

```
Terraform/uat/troubleshoot_ec2_instance/main.tf
Terraform/staging/troubleshoot_ec2_instance/main.tf
```
---
