> [!CAUTION]
> This solution accelerator is not an official Microsoft product! It is a solution accelerator, which can help you implement a monitoring solution within Fabric. As such there is no offical support available and there is a risk that things might break.

# Introduction 

Platform administrators face the challenge of observing the activities within the entire platform. There are multiple sources that provide information, such as capacity events, gateway locks, audit logs, and the platform inventory itself. Additionally, there is a need to obtain this data quickly to observe and react to events automatically.

To address this, a solution has been developed based on Fabric Real-Time Intelligence (RTI). This solution extracts information from sources like the new capacity events available in the Real Time Hub (RTH), audit logs through the API, gateway locks via a script included in the solution, and the inventory of the whole platform through the API.

The following architecture illustrates how the solution interacts with different components to receive, process, store, present dashboards, and react to the platform's information. It also enables longer retention of logs for historical purposes and allows comparison of the current state with previous states to create custom solutions.

![image](/Images/01_PlatformObservabilityArchitecture.png)

This solution uses Microsoft Fabric to address these issues by providing: 

- In-App Configurations
- Real-Time Dashboard
- PowerShell Setup Configurations for the Gateway Events

Benefits include faster incident response, improved health analytics, and streamlined operations, consequently enhancing overall efficiency and reducing downtime. 

# List of items used

The following Fabric items are deployed and used:
- Eventstreams:
   - CapacityUtilizationEvents
      - For the RTH Capacity Events
   - GatewayMonitoringHeartbeat
      - To receive the gateway heartbeat
   - GatewayMonitoringReports
      - To receoive the gateway reports
- Eventhouse:
   - Platform and Audit DB
      - To process, store and query all the information ingested
- Notebooks:
   - Monitoring Audit Logs
      - Extract the Audit logs from the API and ingest them incrementally in the Eventhouse.
      - Recommended to be configured to run every 5 min.
   - Monitoring Extraction Refreshables:
      - Extract the refreshables from all the capacities and stores the last refresh incrementally.
      - Recommended to be configured to run every 5 min.
   - Monitoring Extraction Scanner:
      - Extract the full inventory with the Scanner API.
      - Creates the snapshot and loads incrementally.
      - Recommended to be configured to run every 30 min.
   - Monitoring Extraction Inventory:
      - Extracts the following information at tenant level:
         - Capacities
         - Apps
         - Domains
         - Tenant Settings
         - Workspace Delegated Settings
         - Capacity Delegated Settings
         - Domain Delegated Settings
         - Gateways and Members
         - Gateway Connections
         - Git Connections
      - Recommended to be configured to run every 1 day.


The Notebooks uses the [Semantic Link Labs](https://github.com/microsoft/semantic-link-labs) to interact with the APIs.

# Implementation guide 

## Full process overview 

To implement this solution, we have some step to follow. This steps will cover the creation of all the items in the previous architecture and the script in the Gateway Nodes. We can find the following steps needs to be done: 

- Fabric items initial setup
- Eventstream changes
- Notebook scheduling
- Real-Time Dashboard configuration
- Script deployment and setup in the gateway nodes (Optional)
- Power BI Gateway Report (Optional)


## Requirements and estimated workloads 

- Service Principal - [How to Register an App in Microsoft Entra ID](https://learn.microsoft.com/entra/identity-platform/quickstart-register-app)
- An Entra ID Security Group with the Service Principal as Member
- An Azure Key Vault ([What is Azure Key Vault?](https://learn.microsoft.com/azure/key-vault/general/basic-concepts)) with the following secrets:
   - Tenant ID
   - App (Client) ID of the Service Principal
   - Secret of the Service Principal
- The user that is going to execute the scripts must have at least "Key Vault Secrets User" - [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](https://learn.microsoft.com/azure/key-vault/general/rbac-guide?tabs=azure-cli)
- The Security Group of the service principal must have the following permissions in Fabric Tenant Settings:
   - [Service principals can use Fabric APIs](https://learn.microsoft.com/en-us/fabric/admin/tenant-settings-index#developer-settings)
   - [Service principals can access read-only admin APIs](https://learn.microsoft.com/en-us/fabric/admin/tenant-settings-index#admin-api-settings)
   - [Enhance admin APIs responses with detailed metadata](https://learn.microsoft.com/en-us/fabric/admin/tenant-settings-index#admin-api-settings)
   - [Enhance admin APIs responses with DAX and mashup expressions](https://learn.microsoft.com/en-us/fabric/admin/tenant-settings-index#admin-api-settings)
   - [Member Role over the workspace to use](https://learn.microsoft.com/en-us/fabric/fundamentals/roles-workspaces)
   - [Admin Role over the On-Premise Data Gateways to monitor](https://learn.microsoft.com/en-us/data-integration/gateway/manage-security-roles)
- Microsoft Fabric Capacity of F8 or higher, recommended F16 (the capacity size needed will depend on the amount of logs sent and processed by the system)


## Fabric initial setup 

Create a workspace and import the [Platform Monitoring Setup Notebook](/setup/Platform%20Monitoring%20Setup.ipynb). Follow the instructions for the first run.

If you change the variable "FIRST_RUN" to "False" the script will update the Eventhouse definition and notebooks. 

> [!CAUTION]
> No change are made to any additional item in the workspace or eventhouse. But if you customize the default ones (Notebook, Policies, Tables, Functions, etc), the change could be reverted back or the update could fail.

## Eventstream changes

## Notebook scheduling

## Real-Time Dashboard configuration

## Script deployment and setup in the gateway nodes (Optional)

These are available scripts to retrieve and process logs from on-premises gateways. These scripts help in managing and processing logs. 

You can find the gateway scripts in the subfolder [/gateway/PowerShellScript](/gateway/PowerShellScript)

Requerements:
- PowerShell 7+

### The setup-configuration 

First use the [Gateway Config Notebook](/setup/Gateway%20Config.ipynb) to generate the configuration script for the PowerShell application.

Download the config.json created in the “Built-in Resources” of the notebook and create a folder "/configs/" in the scripts root folder and copy the JSON file.

![image](/Images/02_Notebook_Builtin_Resources_Example.png)

Execute the Setup-UpdateConfiguration Script. The script will first ask you whether you still need to install the necessary PowerShell Modules needed for Lakehouse connectivity (Az.Accounts, Az.Storage, DataGateway).  

Az.Accounts is a module that manages credentials and common configuration for all Azure modules. The Az.Storage module is a PowerShell module that provides cmdlets for managing and interacting with Azure Storage resources. The Data Gateway module is responsible for managing On-premises data gateway and also Power BI data sources. 

 More information about these modules and Az PowerShell can be found here: 

- [Install Azure PowerShell on Windows | Microsoft Learn ](https://learn.microsoft.com/en-us/powershell/azure/install-azps-windows?view=azps-11.6.0&tabs=powershell&pivots=windows-psgallery)
- [Sign in to Azure PowerShell interactively | Microsoft Learn ](https://learn.microsoft.com/en-us/powershell/azure/authenticate-interactive?view=azps-12.0.0)
- [az account | Microsoft Learn ](https://learn.microsoft.com/en-us/cli/azure/account?view=azure-cli-latest)
- [az storage | Microsoft Learn ](https://learn.microsoft.com/en-us/cli/azure/storage?view=azure-cli-latest)
- [PowerShell Cmdlets for On-premises data gateway management | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/gateway/overview?view=datagateway-ps)
- [Use Azure service principals with Azure PowerShell | Microsoft Learn ](https://learn.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-11.6.0)

Once the modules have been set, the script will automatically retrieve the Gateway ID and set up the connections to the Eventstream and Lakehouse.

### Run-GatewayHeartbeat Script:  

The heartbeat logs contain the status of a gateway.  The script will loop and send the logs to the Eventstream to be processed in Fabric. 

### Run-UploadGatewayLogs Script  

This is the main script that does the data movement from local to the service. In case of the Report files we can send the files to the Eventstream and Lakehouse. The log files are sent to the Lakehouse.

### Get-DataGatewayInfo

It will get the Gateway Node info, we can run this once per week or even lower rate.

### Schedule the Scripts

We can use the Task Scheduler in Windows to automate the script. You will fin a template of the Task Schedulers in the folder [\TaskSchedulers](/gateway/TaskSchedulers)


## Power BI Gateway Report (Optional)

Use the template in the [\Gateway Monitoring.pbit](/gateway/PBI%20Report/Gateway%20Monitor.pbit) and put the parameters:
-KustoURL: The Query URL found in the Eventhouse
-KustoDB: The name of the KQL DB

<img width="515" alt="image" src="/Images/11%20-%20PBIT%20Parameters.png">

Then load the report and use the credentials with access to the Eventhouse.

You will find the following pages.

### Gateways

Description of the gateway and the indicator if the heartbeat has been received in the last minute.

<img width="1543" alt="image" src="/Images/13%20-%20Report%20Gateway%20Information.png">

### Jobs

A summary of all jobs (Semantic Model refresh, Dataflow Gen1 refresh, Dataflow Gen2 and Paginated Reports) executed in the clusters.

You can filter by the date you want to look into, how many days you want to look back, the cluster name and the node name.

Selecting a Job in the list will allow you to do a "Drill through" to the Job Details.

<img width="1543" alt="image" src="/Images/14%20-%20Report%20Jobs.png">

### Job Details

The details of the job, where you can find:
- Summary of the queries related to the job
- How many queries has errors, if any
- Data source kinds summary
- Number of queries by node in the cluster for the job
- Service name related to the job (Power BI Datasets (Semantic Models), Power Query Online (Dataflows Gen2), Dataflows and Paginated Reports)
- Workspace ID and Item ID (Dataflows Gen1 don´t provide this information)
- Errors per query, in which node did it happen and the details of the error
- Summary of connections kind and the path
- Query details with the full information of the query

<img width="1508" alt="image" src="/Images/15%20-%20Report%20Job%20Details.png">


### Queries

General information of all queries that ran in the gateways

<img width="1418" alt="image" src="/Images/16%20-%20Report%20Queries.png">

### Running Jobs

Will show only the jobs that are running in the gateways. Selecting a job and going to the details will give you the information on the job and related queries.

<img width="1412" alt="image" src="/Images/17%20-%20Report%20Running%20Jobs.png">

### System Counters

Overview of the system counters report generated by the Gateways.

<img width="1406" alt="image" src="/Images/18%20-%20Report%20System%20Information.png">
