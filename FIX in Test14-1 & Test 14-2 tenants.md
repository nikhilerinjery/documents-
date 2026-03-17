## Enable and map FIX to regression simulator(r19-simulator.redblacksoftware.com)  in Test13-1 & Test 13-2 tenants

##  Configuration Changes
Update the CloudFormation JSON file to point the sender IDs to the Test13 tenants.

```File Path: infra-as-code\CloudFormation\Dev-Mumbai\FIX\Regression\FIX_Service_ECS_Service.json```

## 🔄 Change Details
Locate the REDBLACK_FixConfiguration section and update the values as follows:

```
{
    "Name": "REDBLACK_FixConfiguration__Sessions__2__SocketConnectHost",
    "Value": "r19-simulator.redblacksoftware.com"
},
{
    "Name": "REDBLACK_FixConfiguration__Sessions__2__SenderSubIds__0",
    "Value": "test13-1.redblacksoftware.com"
},
{
    "Name": "REDBLACK_FixConfiguration__Sessions__2__SenderSubIds__1",
    "Value": "test13-2.redblacksoftware.com"
}
```
## 2. Deployment & Verification
After updating the JSON file, apply the CloudFormation template.

Deployment Info
Cluster: dev-mumbai-ecs-linux

Service Name: rb-dev-mumbai-reg-fix-service

✅ Verification Steps
Sometimes the CloudFormation change set may not visually update in the console immediately. Follow these steps to confirm:

Navigate to the ECS Cluster: ```dev-mumbai-ecs-linux.```

Open the Service: ```rb-dev-mumbai-reg-fix-service.```

Go to the Tasks tab and click on the running Task Definition.

Scroll down to Environment Variables and confirm the SenderSubIds show test13-1 and test13-2.


## 3. AppConfig Validation
Ensure the application configuration is correctly set for the Test13 environment.

Go to AWS AppConfig > Applications.

Select ```rb-dev-mumbai-rbcloud-common.```

Select the test13 configuration profile.

Verify the FixOptions block matches the following:

```
"FixOptions": {
    "SenderCompanyId": "REDBLACK",
    "FixApiUri": "https://fixapireg.redblacksoftware.com/",
    "GrpcConnectionPort": 443,
    "GrpcConnectionHost": "fixapigrpcreg.redblacksoftware.com",
    "GrpcSslPemFilePath": "c:\\\\certificate.pem",
    "FixEnabled": "true"
}
```


must contact the requester to ensure the same changes are applied manually to the Hub Database. The FIX connection will fail if the DB and the Infrastructure settings are out of sync.
