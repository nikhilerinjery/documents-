# Restarting a Service Container in Production: Step-by-Step Guide

## 1. Access Jenkins Agent
Before beginning, you must gather the necessary credentials and identifiers from the Production Workspace.

Navigate to the Production Workspace.

Retrieve Identifiers:

Jenkins Instance ID

Jenkins Windows Agent Instance ID

Obtain Credentials:

Jenkins Agent Password

## 2. Connect to Jenkins Agent
The restart commands must be executed from the designated jump box.

Open Remote Desktop Connection (RDP).

Connect to the Jenkins Windows Agent using the credentials obtained in Step 1.

Once the desktop loads, launch PowerShell.

3. Identify API Container IP
To interact with the cluster, you need a stable entry point.

Select and note the IP address of an active API container.

[!IMPORTANT]
Note: Always use the API container IP instead of the service container IP for management commands to avoid unintended impact during service restarts.

## 4. Check Cluster Status
Verify the health and configuration of the cluster before proceeding.

```
pbm <API_IP>:55190 cluster show
```
Expected Outcome:

Displays comprehensive cluster details.

Typical State: Usually lists active clusters (e.g., 6 running clusters).

## 5. Restart the Service
Trigger the service to leave the cluster, which initiates the restart logic.

```
pdm <API_IP>:55190 cluster leave -a akka.ssl.tcp://redblack@<SERVICE_IP>:55100
```

<API_IP>: Replace with the API container IP noted in Step 3.

<SERVICE_IP>: Replace with the IP of the specific service container requiring a restart.

Behavior:

The specified service will attempt to restart.

Delay: If the service does not restart immediately, please wait for approximately 1 minute.

## 6. Identify EC2 Instance
If manual intervention is required, locate the underlying host.

Copy the Service IP.

Navigate to the EC2 Console.

Search using the Service IP to identify the specific instance.

Copy the EC2 Instance ID.

## 7. Locate Container Instance in ECS
Map the EC2 host to the ECS logical container instance.

Navigate to: ECS → Clusters → Production ECS Cluster (Windows).

Go to the Infrastructure section.

Search/Paste the EC2 Instance ID.

Retrieve the corresponding Container Instance ID.

## 8. Stop the Service Task
Manually stopping the task ensures ECS triggers a fresh deployment of the service.

Go to the Tasks section within the ECS cluster.

Filter the list using the Container Instance ID.

Identify the two tasks associated with the instance:

API Task

Service Task

Select the Service Task.

Click Stop.
