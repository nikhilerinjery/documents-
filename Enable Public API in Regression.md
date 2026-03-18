# Enable Public API in Regression

## 2026.1 has been released to regression. Please make the public API changes to enable public API for 2026.1.

---

## Step 1: Update Configuration Files
Navigate to the infrastructure-as-code directory for the target environment.

**File Path:** `/infra-as-code/CloudFormation/Dev-Mumbai/RBCloud/Regression/Reg_ECSServices_parameters.json`

Locate and update the following parameters:

```
        {
                "ParameterKey": "VersionNumberWithOutPatch",
                "ParameterValue": "2025.3.2"
            },
            {
                "ParameterKey": "nlbdomainname",
                "ParameterValue": "2025-3-2-rbcloudapi-regresssion.redblacksoftware.com"
            },
        change to
        {
                "ParameterKey": "VersionNumberWithOutPatch",
                "ParameterValue": "2026.1.0"
            },
            {
                "ParameterKey": "nlbdomainname",
                "ParameterValue": "2026-1-0-rbcloudapi-regression.redblacksoftware.com"
            },

```    
---

## Step 2: Execute CloudFormation via Jenkins
1. **Identify Stack Details:** Go to the ecs `dev-mumbai-ecs-windows-Regression` and retrieve the **Stack Name**, **JSON file**, and **Parameter file** from the tags.
2. **Run Jenkins Job:** /Dev-Mumbai/job/RBCloud_Environment_Setup/.
3. **Review & Apply:** Once the CloudFormation run starts, review the **Change Set**. If everything looks correct, apply it to update the stack.

---

## Step 3: Deploy API Gateway
Once the CloudFormation stack update is complete:
1. Open the **AWS API Gateway** console.
2. Select the API: `rb-reg-public-api-rbcloud` (for Regression).
3. Click **Resources** -> **Deploy API**.
4. Select `reg` from the **Deployment Stage** dropdown.
5. Click **Deploy**.

*Note: Allow 2–3 minutes for the deployment to propagate.*

---

## Step 4: Restart ECS Tasks
To ensure the changes take effect, you must cycle the containers:
1. Open the **Amazon ECS** console.
2. Select the cluster: `dev-mumbai-ecs-windows-Regression`.
3. Locate the **API** and **Service** tasks.
4. **Stop** the existing tasks; ECS will automatically provision new containers with the updated configuration.
