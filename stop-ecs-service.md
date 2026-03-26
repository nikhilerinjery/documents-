🚀 Service Container Restart Guide
🌐 Production Environment | Jenkins + AWS ECS (Windows)
This document provides a fail-safe, step-by-step procedure to restart service containers with minimal production impact.

## 📋 Phase 1: Preparation & Access
Before touching the infrastructure, ensure you have the necessary "keys to the kingdom."

Navigate to: Production Workspace

Identify Identifiers:

Jenkins Instance ID

Jenkins Windows Agent Instance ID

Retrieve Credentials: Obtain the Jenkins Agent Password.

## 🖥️ Phase 2: Establish Connection
All commands must be executed from the secure Jenkins Windows Agent jump box.

Launch RDP: Open Remote Desktop Connection.

Authenticate: Log in to the Windows Agent using the credentials from Phase 1.

Initialize: Open a PowerShell terminal as Administrator.

## 🔍 Phase 3: Cluster Discovery
Identify your targets and verify environment health before execution.

### 1. Identify API IP
Locate and copy the IP address of an active API container.

[!CAUTION]
CRITICAL: Always use the API Container IP (not the Service IP) for management commands. This ensures cluster stability during the restart process.

### 2. Verify Cluster Health
Run the following command to check the current state:

PowerShell
pbm <API_IP>:55190 cluster show
What to look for: A list of active clusters (typically ~6).

Note: If versions 3.1 and 3.2 are both running, you will see multiple cluster entries.

## ⚡ Phase 4: The Restart Execution
This is the core action that triggers the service handoff within the Akka cluster.

Run the leave command:

PowerShell
pdm <API_IP>:55190 cluster leave -a akka.ssl.tcp://redblack@<SERVICE_IP>:55100
<API_IP>: The IP noted in Phase 3.

<SERVICE_IP>: The IP of the specific service container you are restarting.

[!TIP]
Patience is Key: The service may not bounce instantly. Wait at least 60 seconds for the automated logic to trigger before moving to manual intervention.

## 🛠️ Phase 5: Manual Task Management (If Required)
If the cluster leave command does not trigger an automatic restart, follow these steps to force a fresh task in AWS ECS.

### 1. Identify the EC2 Host
Copy the Service IP.

Go to the AWS EC2 Console.

Search by IP to identify the specific host instance.

Copy the EC2 Instance ID.

### 2. Map to ECS Infrastructure
Navigate to: ECS ➔ Clusters ➔ Production ECS Cluster (Windows).

Select the Infrastructure tab.

Filter/Search by your EC2 Instance ID to find the corresponding Container Instance ID.

### 3. Terminate the Task
Go to the Tasks tab within the cluster.

Filter the list using the Container Instance ID.

Identify the Service Task (ensure you ignore the API Task).

Select the task and click 🔴 Stop.
