---

copyright: 
  years:  2022
lastupdated: "2022-06-29"

keywords: high availability, HA, resiliency, regions, zones

services: virtual-servers, vpc, loadbalancer-service

subcollection: ha-infrastructure

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:ui: .ph data-hd-interface="ui"}
{:cli: .ph data-hd-interface="cli"}
{:terraform: .ph data-hd-interface="terraform"}
{:download: .download}
{:important: .important}
{:note: .note}
{:new_window: target="_blank"}

# Customizing a three-tier highly available infrastructure VPC in a single availability zone with automation
{: #create-three-tier-resilient-vpc-saz-modular} 

Highly available and resilient infrastructures for a single availability zone include multiple components and connections. The individual pieces can be created manually with the {{site.data.keyword.cloud}} dashboard, but it is a time-intensive process. To streamline the process, {{site.data.keyword.cloud_notm}} provides several methods to automate deployment by using Terraform as a base. You can run the Terraform scripts yourself, or use the catalog tile or Schematics user interface to run the scripts. The {{site.data.keyword.cloud_notm}} VPC infrastructure that is created uses Intel Xeon CPUs and additional Intel technologies.

All infrastructures have a core set of features that are used. You can customize your infrastructure by adding the features that you want.  

You can select how you want to deploy the architecture:
*   Deploy on {{site.data.keyword.cloud_notm}} - Use the user interface to enter the information for your infrastructure.
*   Run locally - Download script files to your local machine to configure and run.

## Core and Feature modules
{: #saz-core-feature-modules}

For Terraform and Schematics deployments, the components in a single availability zone 3-tier infrastructure are divided into modules so that you can customize your infrastructure to your needs. You receive all of the core components and can turn features components on or off as needed. Modules are not available for the catalog tile deployment.

|Module    |   Resources   |
|-----------|-------------|
|Core     | Resources needed for all single availability zone infrastructures. All resources are configured.  \n - VPC  \n -  Region  \n - Subnet  \n - Security Group  \n - App Load balancer (ALB)  \n - web VSI + host placement groups  \n - App VSI + host placement groups  |
|Features   | The optional features for a single availability zone infrastructure.  You can select as many as you need.  \n - Instance group  \n - VPN  \n - Public Gateway  \n - File  \n - Cloud Object Storage  \n - Public or Private (web LBaaS)|

## What is created
{: #created-vpc-ha-resilient-saz-modular}

A VPC is a private space in IBM Cloud where you can run an isolated environment with custom network policies. The variables that you define are used by the scripts to provision the virtual private cloud infrastructure resources for you. The default values in the scripts are used to create these resources in a single zone: 

|Resource Name      |             Quantity  | 
|-------------------|-----------------------|  
| VPC             |                 01    |     
| Security Groups    |              05    |     
| Subnets      |                    10    |     
| Bastion VSI        |              01    |     
| App VSI            |              03    |     
| web VSI           |               03    |     
| Floating IP     |                 01    |  
| Placement Groups   |              03    |   

The scripts use variables to specify your account information, VSI server information, Load Balancer information, Auto scale, and Anti-Affinity

The scripts automatically install these software packages:

*  PHP
*  Apache
*  Maria DB
*  Word Press Application


## Caveats
{: #limitations-ha-resilient-saz-modular}

The bastion server is important to access other VSIs. 

Don’t delete the bastion server or update its OS image after creation (**Apply plan**) because the bastion server is used to access other servers. If the Bastion server is removed, you lose access to your infrastructure. Get a snapshot of the bastion server for backup and recovery.
{: note}

## Before you begin
{: #before-begin-ha-resilient-saz-modular}

*   If Terraform is not installed on your machine, [Install the Terraform CLI and the {{site.data.keyword.cloud_notm}} Provider plug-in](/docs/ibm-cloud-provider-for-terraform?topic=ibm-cloud-provider-for-terraform-setup_cli). The minimum Terraform version required is v0.14. You also need to install any software that your operating system requires for Terraform, for example Homebrew for Macs. {: terraform}
*   [Create or retrieve an {{site.data.keyword.cloud_notm}} API key](/docs/account?topic=account-userapikey#create_user_key). The API key is used to authenticate with the IBM Cloud platform and to determine your permissions for IBM Cloud services.
*   If you do not know your resource group ID, go to **Manage > Account > Resource Groups** and locate the resource group ID.
*   [Create or retrieve your SSH key name](/docs/ssh-keys?topic=ssh-keys-getting-started-tutorial). The SSH key is used to SSH to the bastion server.
*   Collect the information that you need to specify for the script parameters:

    |Component type   | Parameter values needed |
    |-----------------|--------------------|
    |User-specific information  | - API key  \n - SSH key or keys  \n - Resource Group  \n - IP address or addresses  \n - Your local machine OS \n - Prefix to add to the resource names |
    |Supported OS type  | - Linux  |
    |Zone   | The zone where resources are deployed|
    |Image to use for region VSIs |  - Bastion Server  \n - App Server  \n - Web Server  \n Images are region-specific. To find the image IDs for your region, open the {{site.data.keyword.cloud_notm}} shell and enter `ibmcloud target -r < region_name >` to set your region, then `ibmcloud is images` to see a list of images for that region.|
    |Auto scale App VSI parameters | - Min server  \n - Max servers  \n - CPU threshold  |
    |Auto scale web VSI parameters | - Min server  \n - Max servers  \n - CPU threshold  |
    |Anti-Affinity VSI parameters  | - App and Web - Host spread |

*   For VPN module validation, you need to create an [on-premises-VPN](https://cloud.ibm.com/vpc-ext/network/vpngateways). When you create the VPN, make note of these parameters, you need this information to run the Terraform scripts:
    * vpn_mode = Possible values are policy and route, by default it will be policy.
    * preshared_key = The Key configured on the peer gateway.
    * peer_cidrs = List of peer CIDRs for the creation of VPN connection.
    * peer_gateway_ip = The IP address of the peer VPN gateway

    It is not possible to connect to a VPN gateway through a VPC, so creating an on-premises-VPN helps in connecting to the VPN gateway which we create using the code.
    {: note}
    {: terraform}

*   For DBaaS configuration, you need to edit several files, in addition to the variables file:
    *  Required DBaaS information includes:
        - Admin password - minimum 10 characters, A-Z, a-z, 0-9 
        - The DB endpoint access - either 'private', 'public' or 'public-and-private' (for mysql databases) 
        - Whether to enable auto-scaling in DBaaS - true or false 
    *  There are also optional variables listed and decribed in the variables file that you can modify for your solution. 

*   For file share configuration, you specify information for the file share and its replica:  {: ui}
    *   share_size - size of the file share, min 10 GB max 3200 GB
    *   share_profile - profile for the file share, either tier-3iops, tier-5iops, or tier-10iops
    *   enableFileShare - whether to enable the file share, either true or false
    *   replica_zone - the zone for the replica, should be in the same region as the infrastructure, for example, if the infrastructure is in us-east-1, the replica should be in us-east-2 or us-east-3
    *   replica_profile - should match share_profile, either tier-3iops, tier-5iops, or tier-10iops

## Customizing a three-tier highly available infrastructure VPC in a single availability zone with the catalog tile
{: #create-vpc-saz-catalogtile-modular}
{: ui}

1.	From the IBM Cloud catalog, select **HA Infrastructure Deployment for 3-tier SAZ**. 
2.  Configure your workspace:
    * Enter a name for the workspace.
    * Select a **Resource group**.
    * Select a **Location** for your workspace. The workspace location does not have to match the resource location.
3.  Set the required deployment variables. Review the input parameters and provide values that match your solution. Some parameters have default values. The values that you must specify include:

    |Component type   | Parameter values needed |
    |-----------------|--------------------|
    |User-specific information  | - API key  \n - SSH key name. For example, "ibmcloud_ssh_key_name1,ibmcloud_ssh_key_name2,..."  \n - Resource Group  \n - IP address. Enter the IP address in the format: "public_ip_address1/32,public_ip_address2/32,...". For example, "103.42.91.78/32"  \n - Your local machine OS  \n - Prefix to add to resource names |
    |Zone  | The zone where resources are deployed  |
    |Image ID to use for region VSIs |  -Bastion Server  \n - App Server  \n - Web Server  \n - Db Server  \n Images are region-specific. To find the image IDs for your region, open the {{site.data.keyword.cloud_notm}} shell and enter `ibmcloud target -r < region_name >` to set your region, then `ibmcloud is images` to see a list of images for that region. |

4.  Set the optional deployment variables. These include parameters for optional features.

    |Component type    | Parameter values needed   |
    |------------------|-------------------------|
    |Auto scale App VSI parameters | - Min server  \n - Max servers  \n - CPU threshold  |
    |Auto scale web VSI parameters | - Min server  \n - Max servers  \n - CPU threshold  |
    |VSI spread strategy (host or power spread) | - App spread strategy  \n - web spread strategy  \n - Db spread strategy |
    |File share  |- share_size  \n share_profile  \n enableFileShare  \n replica_zone  \n replica_profile - should match share_profile, either tier-3iops, tier-5iops, or tier-10iops   |

5.  Accept the license agreement and select **Install**.

    During installation you are redirected to the **Jobs** tab of the Schematics page.  

If you need to adjust any of your parameters, go to the **Settings** tab of the Schematics page to see all of the parameters that you can modify. After you modify a parameter, select **Apply plan**.

## Customizing a three-tier highly available infrastructure VPC in a single availability zone with Schematics
{: #create-vpc-saz-schematics-modular}
{: ui}

To use the files in the GitHub repo in Schematics, you need a personal access token. When you create the token, copy it to a text file for future use. When you leave the create token page, you can no longer view the token.  

1.  Go to the [SAZ GitHub repository](https://github.ibm.com/workload-eng-services/ha-saz). 
2.  Select **Settings** from your account icon menu.
3.  Select **Developer settings**.
4.  Select **Personal Access Tokens**.
5.  Select **Generate new token**.
6.  Copy the new token and save it in a text file.

After you have your access token, use Schematics to create your infrastructure.

1.	From the IBM Cloud menu, select [Schematics](https://cloud.ibm.com/schematics/overview).
2.  Click **Workspaces**.
3.	Click **Create workspace**.
4.  On the **Specify template** page:
    * Enter the URL of high availability SAZ repo, https://github.ibm.com/workload-eng-services/ha-saz. 
    * Enter your Personal Access Token.
    * Select the **Terraform version** that is listed in the readme file.
    * Click **Next**.  
5.  On the Workspace details page:
    * Enter a name for the workspace.
    * Select a **Resource group**.
    * Select a **Location** for your workspace. The workspace location does not have to match the resource location.
    * Select **Next**.
6.  Select **Create** to create your workspace. The files from the GitHub repo are loaded in to Schematics. It can take several minutes for the files to load.
7.  On the workspace **Settings** page, in the Input variables section, review the default input variables and provide values that match your solution. Some parameters have default values. Make sure to mark as “sensitive” the parameters that contain sensitive information like API keys. The values that you must specify include:

    |Component type   | Parameter values needed |
    |-----------------|--------------------|
    |User-specific information  | - API key  \n - SSH key name. For example, "ibmcloud_ssh_key_name1,ibmcloud_ssh_key_name2,..."  \n - Resource Group  \n - IP address. Enter the IP address in the format: "public_ip_address1/32,public_ip_address2/32,...". For example, "103.42.91.78/32"  \n - Your local machine OS  \n - Prefix to add to resource names |
    |Region  | Regions where resources are deployed  |
    |OS type for each server  | -Bastion Server  \n - App Server  \n - Web Server  \n - Db Server  |
    |Image to use for region VSIs |  -Bastion Server  \n - App Server  \n - Web Server  \n - Db Server  |
    |Load Balancer | - Domain Name  \n - Region Code  |
    |Auto scale App VSI parameters | - Min server  \n - Max servers  \n - CPU threshold  |
    |Auto scale web VSI parameters | - Min server  \n - Max servers  \n - CPU threshold  |
    |Anti-Affinity VSI parameters  | - App spread strategy  \n - web spread strategy  \n - Db spread strategy |

     For a more detailed description of each of the parameters, check the GitHub repo readme file. 

8.  Click **Save changes**.
9.	On the workspace Settings page, click **Generate plan**. Wait for the plan to complete. It can take several minutes for the plan to complete.
10.	Click **View log** to review the log files of your Terraform execution plan.
11.	Apply your Terraform template by clicking **Apply plan**. It can take 10 - 20 minutes for the apply to complete.
12.	Review the log file to ensure that no errors occurred during the provisioning, modification, or deletion process.

## Customizing a three-tier highly available infrastructure VPC in a single availability zone with Terraform
{: #procedure-ha-resilient-saz-terraform-modular}
{: terraform}

Use these steps to configure the {{site.data.keyword.cloud_notm}} Provider plug-in and use Terraform to create a resilient infrastructure for a 3-tier application.

1.	Create a project folder in your Terraform installation folder, and change directory to your project folder.

    `mkdir myproject && cd myproject`

2.	Copy the files from [vpc-ha-iac GitHub repository](https://github.com/IBM-Cloud/vpc-ha-iac) single-region folder to the project folder that you created in the Terraform installation folder.

3.  Enter your solution-specific information in the `example.userinput.auto.tfvars` file.  Follow the instructions in the file comments. When you are done, remove the `#` at the beginning of each line and save the file as `userinput.auto.tfvars`. If you do not modify and save the `userinput.auto.tfvars`, the script prompts you for the information during the `terraform plan` command.

    You update the information that you collected in the **Before you begin** section:
    * Your {{site.data.keyword.cloud_notm}} API key.
    * Your 32-digit resource group ID (alphanumeric).
    * Your SSH key for the region where you are creating resources.
    * Your public IP address. 
    * Your local machine type, for example, Windows, Linux, or Mac
    * A prefix to use for your resources so that you can easily identify them. 
    * The zone the resources are created in. For example, us-south-1.
    * For each server, Bastion, app, web, db, you enter the:
        * Custom image ID for the server. 
        * OS image currently supported is Linux.
        * The CPU percentage for the App Instance Group
        * The minimum number of servers to have in the App Instance Group.
        * The maximum nuber of server to have in the App Instance Group.
        * The bandwidth per second in GB.
        * The VSI spread strategy for the db server (host or power spread) for anti-affinity. Power spread is recommended here.
        * The VSI spread strategy for the web and app servers (host or power spread) for anti-affinity. Host spread is recommended here if more than 4 are created for each server type.

4.  If you are creating a VPN, enter the VPN information in the `user_input.auto.tfvars` file. You need the:

    * VPN mode -  Possible values are policy and route, by default it will be policy.
    * Preshared key -  The Key configured on the peer gateway.
    * Peer CIDRs - List of peer CIDRs for the creation of VPN connection.
    * Peer gateway IP - The IP address of the peer VPN gateway.

    You must [setup a tunnel](#saz-config-vpn) after the Terraform scripts complete.
    
5.  If you are using DBaaas, modify the `user_input.auto.tfvars` file in the folder. You must enter:
    * Admin password - minimum 10 characters, A-Z, a-z, 0-9 
    * The DB endpoint access - either 'private', 'public' or 'public-and-private' (for mysql databases) 
    * Whether to enable auto-scaling in DBaaS - true or false 

6.  If you are using DBaaS, there are additional parameters that you can modify in the root level `db_variables.tf` file.  These variables are set to defaults and can be modified to match your solution:

    |Parameter | Description|
    |-------|--------|
    |service | The type of Cloud Databases that you want to create. Example: databases-for-mysql, databases-for-mongodb|
    |plan | The name of the service plan that you choose for your instance. All databases use standard. enterprise is supported only for cassandra (databases-for-cassandra) and mongodb(databases-for-mongodb)|
    |db_version | The version of the database to be provisioned. If omitted, the database is created with the most recent major and ||minor version. (For MySQL, 5.7 is the version supported)|
    |members_memory_allocation_mb | The amount of memory in megabytes for the database, split across all members. If not specified, the default setting of the database service is used, which can vary by database type.|
    |members_disk_allocation_mb | The amount of disk space for the database, split across all members. If not specified, the default setting of the database service is used, which can vary by database type.|
    |members_cpu_allocation_count | Enables and allocates the number of specified dedicated cores to your deployment.
    |users | A list of users that you want to create on the database. Multiple blocks are allowed.|
    |Nested scheme for users:||
    ||name - The user ID to add to the database instance. The user ID must be in the range 5 - 32 characters.
    ||password - The password for the user ID. The password must be in the range 10 - 32 characters
    ||auto_scaling - Configure rules to allow your database to automatically increase its resources. Single block of autoscaling is allowed at once.
    |Nested scheme for auto_scaling:||
    |        |Nested scheme for disk:|
    |          |- capacity_enabled - Auto scaling scalar enables or disables the scalar capacity.|
    |          |- free_space_less_than_percent - Auto scaling scalar capacity free space less than percent.|
    |           |- io_above_percent - Auto scaling scalar I/O utilization above percent. -> [Average Memory utilization]|
    |           |- io_over_period - Auto scaling scalar I/O utilization over period. -> [Average Disk IO utilization for a this period]|
    |           |- io_enabled - Auto scaling scalar I/O utilization enabled.|
    |           |- rate_increase_percent - Auto scaling rate increase percent. -> [Scale up by rate_increase_percent]|
    |           |- rate_limit_mb_per_member - Auto scaling rate limit in megabytes per member.-> [Scale up by rate_increase_percent |every rate_period_seconds upto a limit of rate_limit_mb_per_member ]|
    |           |- rate_period_seconds - Auto scaling rate period in seconds. -> [Scale up by rate_increase_percent every rate_period_seconds]|
    |           |- rate_units - Auto scaling rate in units|
    |       |Nested scheme for memory:|
    |           |- io_above_percent - Auto scaling scalar I/O utilization above percent. -> [Average IO utilization]|
    |           |- io_enabled- Auto scaling scalar I/O utilization enabled.|
    |           |- io_over_period - Auto scaling scalar I/O utilization over period. ->[Average IO utilization for a this period]|
    |           |- rate_increase_percent - Auto scaling rate in increase percent. -> [Scale up by rate_increase_percent]|
    |           |- rate_limit_mb_per_member - Auto scaling rate limit in megabytes per member. [Scale up by rate_increase_percent every rate_period_seconds upto rate_limit_mb_per_member]|
    |           |- rate_period_seconds - Auto scaling rate period in seconds. -> [Scale up by rate_increase_percent every |rate_period_seconds]|
    |           |- rate_units - Auto scaling rate in units.|
    |tags |A list of tags that you want to add to your instance.|
    |whitelist | A list of allowed IP addresses for the database. Multiple blocks are allowed.Nested scheme for whitelist:|
    ||- address - The IP address or range of database client addresses to be whitelisted in CIDR format.|
    ||-description - A description for the allowed IP addresses range.|
    ||-backup_id - The CRN of a backup resource to restore from. The backup is created by a database deployment with the same service ID. The backup is loaded after provisioning and the new deployment starts up that uses that data.|
    |backup_encryption_key_crn | The CRN of a key protect key, that you want to use for encrypting disk that holds deployment backups.|
    |key_protect_key | The root key CRN of a Key Management Services like Key Protect or Hyper Protect Crypto Service (HPCS) that you want to use for disk encryption.|
    |key_protect_instance | The instance CRN of a Key Management Services like Key Protect or Hyper Protect Crypto Service (HPCS) that you want to use for disk encryption.|
    |remote_leader_id | A CRN of the leader database to make the replica(read-only) deployment. The leader database is created by a database deployment with the same service ID. A read-only replica is set up to replicate all of your data from the leader deployment to the replica deployment by using asynchronous replication.|

7.	Initialize the Terraform CLI. 

    `terraform init`

8.	Create a Terraform execution plan. The Terraform execution plan summarizes all the actions that are done to create the virtual private cloud instance in your account.

    `terraform plan`

    The scripts prompt you for information about your resources if you did not specify them in the `userinput.auto.tfvars` file.  

9.	Verify that the plan shows all of the resources that you want to create and that the names and values are correct. If the plan needs to be adjusted, edit the `userinput.auto.tfvars` and `modules .tf` file and run `terraform plan` again.

10.	Run the terraform script by using the below command:

    `terraform apply`

    The virtual private cloud and components are created and you see output similar to the terraform plan output. 

## Configure the VPN (optional)
{: #saz-config-vpn}
{: terraform}

There will be a connection created in [prefix]VPN (status-down). You need to create a connection in the On-premises-VPN to make a tunnel between the two VPNs. 

### Create the tunnel 

1. Go to on-prem-VPN that you have before running the scripts. Click VPN connection and then **Create**.
2. Update these values:

   a. VPN connection name : Any unique name
   b. Peer gateway address : saz-vpn Gateway IP
   c. Preshared key : < password that you created when you created the on-prem VPN >
   d. Local IBM CIDRs : On-prem-vpn subnet ip-range. 
      Go to the saz-vpn connection, copy the Peer CIDRs value and paste.
   e. Peer CIDRs. : saz-vpn local IBM CIDR. 
      Go to saz-vpn connection, copy the Local IBM CIDRs and paste.
3. Click **Create VPN** connection.

### Make the connection active by setting up the Allowlist:

1. Go to **Bastion VSI > Security groups > Rules > Manage rules > Create Inbound rule**. The Inbound rule port range value for both min and max is 22.
2. After **Allowlisting**, go back to both the connection pages and refresh. Make sure the connections are in the **Active** state.

### Set SSH Key

Use the `ssh` command, the connection starts from your local machine to on-prem VSI and from on-prem VSI to the bastion VSI.

Your public key is in Bastion, and you need to copy your private key to the on-prem VSI to be able to ssh into the bastion VSI.
{: note}

## Next Steps
{: terraform}

If you need to rename your resources after they are created, modify the `example.userinput.auto.tfvars` and `modules.tf` file to change the names and run terraform plan and terraform apply again. 

Do not use the {{site.data.keyword.cloud_notm}} Dashboard and user interface to modify your VPC after it is created. The Terraform scripts create a complete solution and selectively modifying resources with the user interface might cause unexpected results. 

If you need to remove your VPC, use `terraform destroy`. The Bastion server is protected for inadvertent deletion. Before you run the `terraform destroy` command, you need to set the `prevent_destroy` flag to `false` to remove your VPC. The `prevent_destroy` flag is located in `./modules/bastion/compute.tf`.
