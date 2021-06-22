---
layout: post
title: Monitor syslog on an Azure Arc enabled Linux server
---

This blog describes how to use Azure Arc to monitor syslog from an Ubuntu server running outside of Azure. I use an [Arc Connected Machine](https://docs.microsoft.com/en-us/azure/azure-arc/servers/overview) configured with the [Log Analytics extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/oms-linux) to [collect syslog events](https://docs.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-syslog) and send them into a Log Analytics Workspace on Azure.

Prerequisites: If you don't already have it, [install azure-cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt) on the remote machine you're going to monitor.

Except for getting a security key in Step 7, and viewing the final results in Step 9, all of the commands below should be run on the remote machine.


## 1. Register the necessary resource providers in your Azure subscription
Add the necessary Resource providers into an Azure subscription:

```
az provider register --namespace 'Microsoft.GuestConfiguration'
az provider register --namespace 'Microsoft.Microsoft.HybridCompute'
az provider register --namespace 'Microsoft.Microsoft.OperationalInsights'
```

Verify they've been registered:

```
az provider list --query "[?namespace=='Microsoft.GuestConfiguration' || namespace=='Microsoft.HybridCompute'].{Provider: namespace, Status:registrationState}" --out table
```


## 2. Install Azure Connected Machine Agent
Download and run install script, which fetches and installs the Azure Connected Machine Agent (azcmagent)

```
wget https://aka.ms/azcmagent -O install_connected_machine_agent.sh
chmod +x install_connected_machine_agent.sh && ./install_connected_machine_agent.sh
```

If it's successfully installed, the last line will be "Latest version of azcmagent is installed."


## 3. Get Subscription and Tenant ID.

Look up your subscription ID and tenant ID, either from the Portal (Step 3A) or from the CLI (Step 3B).
### 3a. Via Portal
For Tenant ID, go to Azure Active Directory service, and copy the Tenant ID from the screen.
For Subscription ID, go to Subscriptions service, and copy the Subscription ID from the screen.

### 3b. Via CLI
```
az account tenant list --query '[].tenantId'
az account subscription list --query '[].subscriptionId'
```

If this is the first time you've run az account, you'll get Y/N prompt:
The command requires the extension account. Do you want to install it now? The command will continue to run after the extension is installed. (Y/n):

## 4. Create Resource Group
Create a resource group to hold your connected machine(s).
```
az group create -l eastus -n MyConnectedMachines001
```

## 5. Onboard the machine to Azure Arc
You'll need your tenant ID, subscription ID, region, and resource group that you created above for this step.

```
sudo azcmagent connect --resource-group "MyConnectedMachines001" --tenant-id "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" --location "eastus" --subscription-id "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz"
```

![Connect Machine to Arc]({{ site.baseurl }}/images/03-ConnectMachineToArc.png "Connect Machine to Arc")

During this onboarding, you'll be given a link and a code to authenticate via a browser (see pictures).

![Open Link and enter code]({{ site.baseurl }}/images/03-EnterCode.png "Authenticate using browser")

Verify that the machine has been connected to Arc:
```
az connectedmachine list
```

This should produce JSON showing details on the connnected server.  Look for the key/value pair of "status" : "Connected" towards the end of the JSON output.

You can also view the machine details in the Azure Portal now, under Azure Arc. Click on "Servers" under the "Infrastructure" heading.


## 6. Create a Log Analytics Workspace

Create a Log Analytics Workspace to hold the connected machine's log files.
```
az monitor log-analytics workspace create --resource-group "MyConnectedMachines001" \
--workspace-name "ConnectedMachineWorkspace001" --location "eastus" \
--subscription "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz"
```

This will take a couple of minutes to create.

## 7. Add the Log Analytics extension to the connected machine

Now add the "Log Analytics Agent - Azure Arc" extension to the machine. For this step, you will need the secret WorkspaceKey, which you can retrieve from the Azure Portal.

In the Azure Portal, go to the Log Analytics Workspaces service.  Click on the name of the workspace you just created ("ConnectedMachineWorkspace001").  Click on Agents Management (on the left navigation pane).
Copy the Workspace ID and the Primary Key.

On the machine you're connecting, run this command, using the ID and Key you copied above:

```
az connectedmachine extension create --machine-name "penguin" --name "OmsAgentForLinux" \
--location "eastus" --settings '{"workspaceId":"XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXX"}' \
--protected-settings '{"workspaceKey":"YYYYYYYYYYYYYYYYYYYYY"}' \
--resource-group "MyConnectedMachines001" --type-handler-version "1.13" \
--type "OmsAgentForLinux" --publisher "Microsoft.EnterpriseCloud.Monitoring" 
```

This will take several minutes to execute.  (It's installing the OMS agent on your local machine.)

Once done, you can verify that the extension has been installed locally by running
```
az connectedmachine extension list --machine-name "penguin" --resource-group "MyConnectedMachines001"
```


## 8. Tell the Azure agents what syslog events to collect
The Log Analytics extension installs the OMS agent on your connected machine. When running on a connected machine, the default it to collect no events.  Edit the OMS agent config file, and have it report some events.  A full list of subsystems that the OMS agent can monitor is [here](https://docs.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-syslog).

Edit /etc/rsyslog.d/95-omsagent.conf, and add the content below to this file.
```
sudo nano /etc/rsyslog.d/95-omsagent.conf
```

add:
# OMS Syslog collection for workspace a83ef1f4-c430-4fce-925d-e05387c6b193
```
kern.warning       @127.0.0.1:25224
daemon.warning     @127.0.0.1:25224
syslog.warning     @127.0.0.1:25224
cron.info          @127.0.0.1:25224
```

Save the file, then restart rsyslog with
```
sudo service rsyslog restart
```


## 9. View the connected machine's syslog events in Azure Arc

In the Azure Portal, go to Log Analytics Workspaces.  Click on ConnectedMachineWorkspace001.
On the left-hand navigation bar, select Logs.
Close the Queries window ("X" in the upper right corner). 
In the query text box, enter

`Syslog 
| top 100 by TimeGenerated desc
`
Click Run, and browse the logs.
![Browse syslog from Query results]({{ site.baseurl }}/images/03-ViewSyslogEventsFromAzurePortal.png "Browse syslogs from Query results")



## 10. Clean up

Delete the Log Analytics Extension from the connected machine:
```
az connectedmachine extension delete --machine-name "penguin" --name "OmsAgentForLinux" --resource-group "MyConnectedMachines001"
```

Disconnect the connected machine from Azure Arc:
```
sudo azcmagent disconnect
```

delete the Azure Connected Machine Agent from your machine:
```
sudo apt-get purge azcmagent
```

Delete the guest configuration (gc_linux_service)
```
sudo apt-get purge packages-microsoft-prod
```

Delete the resource group
```
az group delete -n MyConnectedMachines001
```
