## Exercise 3: Migrate the application and web tiers using Azure Migrate: Server Migration

Duration: 90 minutes

In this exercise you will migrate the web tier and application tiers of the application from on-premises to Azure using Azure Migrate: Server Migration.

Having migrated the virtual machines, you will reconfigure the application tier to use the application database hosted in Azure SQL. This will enable you to verify that the migration application is working end-to-end.

### Task 1: Create a Storage Account

In this task you will create a new Azure Storage Account that will be used by Azure Migrate: Server Migration for storage of your virtual machine data during migration.

> **Note:** This lab focuses on the technical tools required for workload migration. In a real-world scenario, more consideration should go into the long-term plan prior to migrating assets. The landing zone required to host VMs should also include considerations for network traffic, access control, resource organization, and governance. For example, the CAF Migration Blueprint and CAF Foundation Blueprint can be used to deploy a pre-defined landing zone, and demonstrate the potential of an Infrastructure as Code (IaC) approach to infrastructure resource management. For more information, see [Azure Landing Zones](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/) and [Cloud Adoption Framework Azure Migration landing zone Blueprint sample](https://docs.microsoft.com/azure/governance/blueprints/samples/caf-migrate-landing-zone/).

1. In the Azure portal's left navigation, select **+ Create a resource**, then search for and select **Storage account**, followed by **Create**.

    ![Screenshot of the Azure portal showing the create storage account navigation.](images/Exercise3/create-storage-1.png "Storage account - Create")

2. In the **Create storage account** blade, on the **Basics** tab, use the following values:

    - Subscription: **Select your Azure subscription**.
  
    - Resource group: **AzureMigrateRG**
  
    - Storage account name: **migrationstorageSUFFIX**
  
    - Location: **IMPORTANT: Select the same location as your Azure SQL Database** (can be found in the Azure portal).
  
    - Account kind: **Storage (general purpose v1)** (do not use a v2 account).
  
    - Replication: **Locally-redundant storage (LRS)**

    ![Screenshot of the Azure portal showing the create storage account blade.](images/Exercise3/create-storage-2.png "Storage account settings")

3. Select **Review + create**, then select **Create**.

#### Task summary 

In this task you created a new Azure Storage Account that will be used by Azure Migrate: Server Migration.

### Task 2: Create a Virtual Network

In this task you will create a new virtual network that will be used by your migrated virtual machines when they are migrated to Azure. (Azure Migrate will only create the VMs, their network interfaces, and their disks; all other resources must be staged in advance.)

> **Note:** Azure provides several options for deploying the right network configuration. After the lab, if you’d like to better understand your networking options, see the [network decision guide](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/software-defined-network/), which builds on the Cloud Adoption Framework’s Azure landing zones. 

You will also configure a private endpoint in this network to allow private, secure access to the SQL Database.

1. In the Azure portal's left navigation, select **+ Create a resource**, then select **Networking**, followed by **Virtual network**.

    ![Screenshot of the Azure portal showing the create virtual network navigation.](images/Exercise3/create-vnet-1.png "New Virtual Network")

2. In the **Create virtual network** blade, enter the following values:

    - Subscription: **Select your Azure subscription**.
  
    - Resource group: (select existing) **SmartHotelRG**
  
    - Name: **SmartHotelVNet**
  
    - Region: **IMPORTANT: Select the same location as your Azure SQL Database**.

    ![Screenshot of the Azure portal showing the create virtual network blade 'Basics' tab.](images/Exercise3/create-vnet-2.png "Create Virtual Network - Basics")

3. Select **Next: IP Addresses >**, and enter the following configuration. Then select **Review + create**, then **Create**.

    - IPv4 address space: **192.168.0.0/24** 
  
    - First subnet: Select **Add subnet** and enter the following then select **Add**

        - Subnet name: **SmartHotel**
   
        - Address range: **192.168.0.0/25**
  
    - Second subnet: Select **Add subnet** and enter the following then select **Add**. 

        - Subnet name: **SmartHotelDB**
   
        - Address range: **192.168.0.128/25**

    ![Screenshot of the Azure portal showing the create virtual network blade 'IP Addresses' tab.](images/Exercise3/create-vnet-3.png "Create Virtual Network - IP Addresses")

4. Navigate to the **SmartHotelDBRG** resource group, and then to the database server. Under **Security**, select **Private endpoint connections**, then select **+ Private endpoint**.

5. On the **Basics** tab, enter the following configuration then select **Next: Resource**:

    - Resource group: **SmartHotelDBRG**
  
    - Name: **SmartHotel-DB-Endpoint**
  
    - Region: **Select the same location as the SmartHotelVNet**.
  
    ![Screenshot showing the 'Create a private endpoint' blade, 'Basics' tab.](images/Exercise3/private-endpoint-1.png "Create a Private Endpoint - Basics")

6.  On the **Resource** tab, enter the following configuration then select **Next: Configuration**:

    - Connection method: **Connect to an Azure resource in my directory**.
  
    - Subscription: **Select your subscription**.
  
    - Resource type: **Microsoft.Sql/servers**
  
    - Resource: **Your SQL database server**.
  
    - Target sub-resource: **sqlServer**

    ![Screenshot showing the 'Create a private endpoint' blade, 'Resource' tab.](images/Exercise3/private-endpoint-2.png "Create a Private Endpoint - Resource")
   
7.  On the **Configuration** tab, enter the following configuration then select **Review + Create** then **Create**:

    - Virtual network: **SmartHotelVNet**
  
    - Subnet: **SmartHotelDB (192.168.0.128/25)**
  
    - Integrate with private DNS zone: **Yes**
  
    - Private DNS zone: (default) **privatelink.database.windows.net**

    ![Screenshot showing the 'Create a private endpoint' blade, 'Configuration' tab.](images/Exercise3/private-endpoint-3.png "Create a Private Endpoint - Configuration")

8. **Wait** for the deployment to complete. Open the Private Endpoint blade, and note that the FQDN for the endpoint is listed as **\<your database\>.database.windows.net**, with an internal IP address **192.168.0.132**.

    > **Note**: Private DNS is used so that the database domain name, **\<your server\>.database.windows.net** resolves to the internal private endpoint IP address **192.168.0.132** when resolved from the SmartHotelVNet, but resolves to the Internet-facing IP address of the database server when resolved from outside the VNet. This means the same connection string (which contains the domain name) can be used in both cases.

#### Task summary 

In this task you created a new virtual network that will be used by your virtual machines when they are migrated to Azure. You also created a private endpoint in this network, which will be used to access the SQL database.

### Task 3: Register the Hyper-V Host with Azure Migrate: Server Migration

In this task, you will register your Hyper-V host with the Azure Migrate: Server Migration service. This service uses Azure Site Recovery as the underlying migration engine. As part of the registration process, you will deploy the Azure Site Recovery Provider on your Hyper-V host.

1. Return to the **Azure Migrate** blade in the Azure Portal, and select **Servers** under **Migration goals** on the left. Under **Migration Tools**, select **Discover**.

    **Note:** You may need to add the migration tool yourself by following the link below the **Migration Tools** section, selecting **Azure Migrate: Server Migration**, then selecting **Add tool(s)**. 

    ![Screenshot of the Azure portal showing the 'Discover' button on the Azure Migrate Server Migration panel.](images/Exercise3/discover-1.png "Azure Migrate: Server Migration - Discover")

2. In the **Discover machines** panel, under **Are your machines virtualized**, select **Yes, with Hyper-V**. Under **Target region** enter **the same region as used for your Azure SQL Database** which can be found in the Azure portal and check the confirmation checkbox. Select **Create resources** to begin the deployment of the Azure Site Recovery resource used by Azure Migrate: Server Migration for Hyper-V migrations.

    ![Screenshot of the Azure portal showing the 'Discover machines' panel from Azure Migrate.](images/Exercise3/discover-2.png "Discover machines - source hypervisor and target region")

    Once deployment is complete, the 'Discover machines' panel should be updated with additional instructions.
  
3. Copy the **Download** link for the Hyper-V replication provider software installer to your clipboard.

    ![Screenshot of the Discover machines' panel from Azure Migrate, highlighting the download link for the Hyper-V replication provider software installer.](images/Exercise3/discover-3.png "Replication provider download link")

4. Launch **Microsoft Edge** from the desktop shortcut, and paste the link into a new browser tab to download the Azure Site Recovery provider installer.

5. Return to the **Discover machines** page in your browser and select the blue **Download** button and download the registration key file.

    ![Screenshot of the Discover machines' panel from Azure Migrate, highlighting the download link Hyper-V registration key file.](images/Exercise3/discover-4.png "Download registration key file")

6. Open the **AzureSiteRecoveryProvider.exe** installer you downloaded a moment ago. On the **Microsoft Update** tab, select **Off** and select **Next**. Accept the default installation location and select **Install**.

    ![Screenshot of the ASR provider installer.](images/Exercise3/asr-provider-install.png "Azure Site Recovery Provider Setup")

7. When the installation has completed select **Register**. Browse to the location of the key file you downloaded. When the key is loaded select **Next**.

    ![Screenshot of the ASR provider registration settings.](images/Exercise3/asr-registration.png "Key file registration")

8.  Select **Connect directly to Azure Site Recovery without a proxy server** and select **Next**. The registration of the Hyper-V host with Azure Site Recovery will begin.

9. Wait for registration to complete (this may take several minutes). Then select **Finish**.

    ![Screenshot of the ASR provider showing successful registration.](images/Exercise3/asr-registered.png "Registration complete")

10. Return to the Azure Migrate browser window. **Refresh** your browser, then re-open the **Discover machines** panel by selecting **Discover** under **Azure Migrate: Server Migration** and selecting **Yes, with Hyper-V** for **Are your machines virtualized?**.

11. Select **Finalize registration**, which should now be enabled.

    ![Screenshot of the Discover machines' panel from Azure Migrate, highlighting the download link Hyper-V registration key file.](images/Exercise3/discover-5.png "Finalize registration")

12. Azure Migrate will now complete the registration with the Hyper-V host. **Wait** for the registration to complete. This may take several minutes.

    ![Screenshot of the 'Discover machines' panel from Azure Migrate, showing the 'Finalizing registration...' message.](images/Exercise3/discover-6.png "Finalizing registration...")

13. Once the registration is complete, close the **Discover machines** panel.

    ![Screenshot of the 'Discover machines' panel from Azure Migrate, showing the 'Registration finalized' message.](images/Exercise3/discover-7.png "Registration finalized")

14. The **Azure Migrate: Server Migration** panel should now show 5 discovered servers.

    ![Screenshot of the 'Azure Migrate - Servers' blade showing 6 discovered servers under 'Azure Migrate: Server Migration'.](images/Exercise3/discover-8.png "Discovered servers")

#### Task summary 

In this task you registered your Hyper-V host with the Azure Migrate Server Migration service.

### Task 4: Enable Replication from Hyper-V to Azure Migrate

In this task, you will configure and enable the replication of your on-premises virtual machines from Hyper-V to the Azure Migrate Server Migration service.

1. Under **Azure Migrate: Server Migration**, select **Replicate**. This opens the **Replicate** wizard.

    ![Screenshot highlighting the 'Replicate' button in the 'Azure Migrate: Server Migration' panel of the Azure Migrate - Servers blade.](images/Exercise3/replicate-1.png "Replicate link")

2. In the **Source settings** tab, under **Are your machines virtualized?**, select **Yes, with Hyper-V** from the drop-down. Then select **Next**.

    ![Screenshot of the 'Source settings' tab of the 'Replicate' wizard in Azure Migrate Server Migration. Hyper-V replication is selected.](images/Exercise3/replicate-2.png "Replicate - Source settings")

3. In the **Virtual machines** tab, under **Import migration settings from an assessment**, select **Yes, apply migration settings from an Azure Migrate assessment**. Select the **SmartHotelVMs** VM group and the **SmartHotelAssessment** migration assessment.

    ![Screenshot of the 'Virtual machines' tab of the 'Replicate' wizard in Azure Migrate Server Migration. The Azure Migrate assessment created earlier is selected.](images/Exercise3/replicate-3.png "Replicate - Virtual machines")

4. The **Virtual machines** tab should now show the virtual machines included in the assessment. Select the **UbuntuWAF**, **smarthotelweb1**, and **smarthotelweb2** virtual machines, then select **Next**.

    ![Screenshot of the 'Virtual machines' tab of the 'Replicate' wizard in Azure Migrate Server Migration. The UbuntuWAF, smarthotelweb1, and smarthotelweb2 machines are selected.](images/Exercise3/replicate-4.png "Replicate - Virtual machines")

5. In the **Target settings** tab, select your subscription and the existing **SmartHotelRG** resource group. Under **Replication storage account** select the **migrationstorageSUFFIX** storage account and under **Virtual Network** select **SmartHotelVNet**. Under **Subnet** select **SmartHotel**. Select **Next**.

    ![Screenshot of the 'Target settings' tab of the 'Replicate' wizard in Azure Migrate Server Migration. The resource group, storage account and virtual network created earlier in this exercise are selected.](images/Exercise3/replicate-5.png "Replicate - Target settings")

    > **Note:** For simplicity, in this lab you will not configure the migrated VMs for high availability, since each application tier is implemented using a single VM.

6. In the **Compute** tab, select the **Standard_F2s_v2** VM size for each virtual machine. Select the **Windows** operating system for the **smarthotelweb** virtual machines and the **Linux** operating system for the **UbuntuWAF** virtual machine. Select **Next**. 

    > **Note**: If you are using an Azure Pass subscription, your subscription may not have a quota allocated for FSv2 virtual machines. In this case, use **DS2_v2 or D2s_v3** virtual machines instead.

    ![Screenshot of the 'Compute' tab of the 'Replicate' wizard in Azure Migrate Server Migration. Each VM is configured to use a Standard_F2s_v2 SKU, and has the OS Type specified.](images/Exercise3/replicate-6.png "Replicate - Compute")

7. In the **Disks** tab, review the settings but do not make any changes. Select **Next**, then select **Replicate** to start the server replication.

8. In the **Azure Migrate - Servers** blade, under **Azure Migrate: Server Migration**, select the **Overview** button.

    ![Screenshot of the 'Azure Migrate - Servers' blade with the 'Overview' button in the 'Azure Migrate: Server Migration' panel highlighted.](images/Exercise3/replicate-7.png "Overview link")

9. Confirm that the 3 machines are replicating.

    ![Screenshot of the 'Azure Migrate: Server Migration' overview blade showing the replication state as 'Healthy' for 3 servers.](images/Exercise3/replicate-8.png "Replication summary")

10. Select **Replicating Machines** under **Manage** on the left.  Select **Refresh** occasionally and wait until all three machines have a **Protected** status, which shows the initial replication is complete. This will take several minutes.

    ![Screenshot of the 'Azure Migrate: Server Migration - Replicating machines' blade showing the replication status as 'Protected' for all 3 servers.](images/Exercise3/replicate-9.png "Replication status")

#### Task summary 

In this task you enabled replication from the Hyper-V host to Azure Migrate, and configured the replicated VM size in Azure.

### Task 5: Configure static internal IP addresses for each VM

In this task you will modify the settings for each replicated VM to use a static private IP address that matches the on-premises IP addresses for that machine.

1. Still using the **Azure Migrate: Server Migration - Replicating machines** blade, select the **smarthotelweb1** virtual machine. This opens a detailed migration and replication blade for this machine. Take a moment to study this information.

    ![Screenshot from the 'Azure Migrate: Server Migration - Replicating machines' blade with the smarthotelweb1 machine highlighted.](images/Exercise3/config-0.png "Replicating machines")

2. Select **Compute and Network** under **General** on the left, then select **Edit**.

   ![Screenshot of the smarthotelweb1 blade with the 'Compute and Network' and 'Edit' links highlighted.](images/Exercise3/config-1.png "Edit Compute and Network settings")

3. Confirm that the VM is configured to use the **F2s_v2** VM size (or **DS2_v2 or D2s_v3** if using an Azure Pass subscription) and that **Use managed disks** is set to **Yes**.

4. Under **Network Interfaces**, select **InternalNATSwitch** to open the network interface settings.

    ![Screenshot showing the link to edit the network interface settings for a replicated VM.](images/Exercise3/nic.png "Network Interface settings link")

5. Change the **Private IP address** to **192.168.0.4**.

    ![Screenshot showing a private IP address being configured for a replicated VM in ASR.](images/Exercise3/private-ip.png "Network interface - static private IP address")

6. Select **OK** to close the network interface settings blade, then **Save** the **smarthotelweb1** settings.

7. Repeat these steps to configure the private IP address for the other VMs.
 
    - For **smarthotelweb2** use private IP address **192.168.0.5**
  
    - For **UbuntuWAF** use private IP address **192.168.0.8**

#### Task summary

In this task you modified the settings for each replicated VM to use a static private IP address that matches the on-premises IP addresses for that machine

> **Note**: Azure Migrate makes a "best guess" at the VM settings, but you have full control over the settings of migrated items. In this case, setting a static private IP address ensures the virtual machines in Azure retain the same IPs they had on-premises, which avoids having to reconfigure the VMs during migration (for example, by editing web.config files).

### Task 6: Server migration

In this task you will perform a migration of the UbuntuWAF, smarthotelweb1, and smarthotelweb2 machines to Azure.

> **Note**: In a real-world scenario, you would perform a test migration before the final migration. To save time, you will skip the test migration in this lab. The test migration process is very similar to the final migration.

1. Return to the **Azure Migrate: Server Migration** overview blade. Under **Step 3: Migrate**, select **Migrate**.

    ![Screenshot of the 'Azure Migrate: Server Migration' overview blade, with the 'Migrate' button highlighted.](images/Exercise3/migrate-1.png "Replication summary")

2. On the **Migrate** blade, select the 3 virtual machines then select **Migrate** to start the migration process.

    ![Screenshot of the 'Migrate' blade, with 3 machines selected and the 'Migrate' button highlighted.](images/Exercise3/migrate-2.png "Migrate - VM selection")

    > **Note**: You can optionally choose whether the on-premises virtual machines should be automatically shut down before migration to minimize data loss. Either setting will work for this lab.

3. The migration process will start.

    ![Screenshot showing 3 VM migration notifications.](images/Exercise3/migrate-3.png "Migration started notifications")

4. To monitor progress, select **Jobs** under **Manage** on the left and review the status of the three **Planned failover** jobs.

    ![Screenshot showing the **Jobs* link and a jobs list with 3 in-progress 'Planned failover' jobs.](images/Exercise3/migrate-4.png "Migration jobs")

5. **Wait** until all three **Planned failover** jobs show a **Status** of **Successful**. You should not need to refresh your browser. This could take up to 15 minutes.

    ![Screenshot showing the **Jobs* link and a jobs list with all 'Planned failover' jobs successful.](images/Exercise3/migrate-5.png "Migration status")

6. Navigate to the **SmartHotelRG** resource group and check that the VM, network interface, and disk resources have been created for each of the virtual machines being migrated.

   ![Screenshot showing resources created by the test failover (VMs, disks, and network interfaces).](images/Exercise3/migrate-6.png "Migrated resources")

#### Task summary

In this task you used Azure Migrate to create Azure VMs using the settings you have configured, and the data replicated from the Hyper-V machines. This migrated your on-premises VMs to Azure.

### Task 7: Enable Azure Bastion

We will need to access our newly-migrated virtual machines to make some configuration changes. However, the machines do not currently have public IP addresses. Rather than add public IP addresses, we will access them using Azure Bastion.

Azure Bastion requires a dedicated subnet within the same virtual network as the virtual machines. Unfortunately, our SmartHotelVNet does not have any free network space available. To address this, we will first extend the network space.

1. Navigate to the **SmartHotelVNet** virtual network, then select **Address space** under **Settings** on the left.  Add the address space **10.10.0.0/24**, and **Save**.

2. Select **Subnets** under **Settings** on the left, and add a new subnet named **AzureBastionSubnet**, with address space **10.10.0.0/27**.

3. Select **+ Create a resource** in the portal's left navigation, then search for and select **Bastion**, then select **Create**.

4. Fill in the **Create a Bastion** blade as follows:

    - Subscription: **Your subscription**
  
    - Resource group: (select existing) **BastionRG**
  
    - Name: **SmartHotelBastion**
  
    - Region: **Same as SmartHotelVNet**
  
    - Virtual Network: **SmartHotelVNet**
  
    - Subnet: **AzureBastionSubnet**
  
    - Public IP address: (Create new) **Bastion-IP**

    ![Screenshot showing the 'Create a Bastion' blade.](images/Exercise3/bastion-create.png "Create a Bastion")

5. Select **Review + create**, then **Create**.

6. **Wait** for the Bastion to be deployed. This will take several minutes.

### Task 8: Configure the database connection

The application tier machine **smarthotelweb2** is configured to connect to the application database running on the **smarthotelsql** machine.

On the migrated VM **smarthotelweb2**, this configuration needs to be updated to use the Azure SQL Database instead.

> **Note**: You do not need to update any configuration files on **smarthotelweb1** or the **UbuntuWAF** VMs, since the migration has preserved the private IP addresses of all virtual machines they connect with.

1. Navigate to the **smarthotelweb2** VM overview blade, and select **Connect**. Select **Bastion** and connect to the machine with the username **Administrator** and the password **demo!pass123**. When prompted, **Allow** clipboard access.

    **Note:** You may have to wait a few minutes and refresh to have the option to enter the credentials. 

    ![Screenshot showing the Azure Bastion connection blade.](images/Exercise3/web2-connect.png "Connect using Bastion")

2. In the **smarthotelweb2** remote desktop session, open Windows Explorer and navigate to the **C:\\inetpub\\SmartHotel.Registration.Wcf** folder. Double-select the **Web.config** file and open with Notepad.

3. Update the **DefaultConnection** setting to connect to your Azure SQL Database.

    You can find the connection string for the Azure SQL Database in the Azure portal by browsing to the database, and selecting **Show database connection strings**.

     ![Screenshot showing the 'Show database connection strings' link for an Azure SQL Database.](images/Exercise3/show-connection-strings.png "Show database connection strings")

    Copy the **ADO.NET** connection string, and paste into the web.config file on **smarthotelweb2**, replacing the existing connection string.  **Be careful not to overwrite the 'providerName' parameter which is specified after the connection string.**

    > **Note:** You may need to open the clipboard panel on the left-hand edge of the Bastion window, paste the connection string there, and then paste into the VM.

    Set the password in the connection string to **demo!pass123**.

    ![Screenshot showing the user ID and Password in the web.config database connection string.](images/Exercise3/web2-connection-string.png "web.config")

4. **Save** the `web.config` file and exit your Bastion remote desktop session.

#### Task summary 

In this task, you updated the **smarthotelweb2** configuration to connect to the Azure SQL Database.

### Task 9: Configure the public IP address and test the SmartHotel application

In this task, you will associate a public IP address with the UbuntuWAF VM. This will allow you to verify that the SmartHotel application is running successfully in Azure.

1. Navigate to the **UbuntuWAF** VM blade, select **Networking** under **Settings** on the left, then select the network interface (in bold text). 

    ![Screenshot showing the path to the NIC of the UbuntuWAF VM.](images/Exercise3/waf-nic.png "Network interface link")

2. Select **IP configuration** under **Settings** on the left, then select the IP configuration listed.

    ![Screenshot showing the path to the ipConfig of the UbuntuWAF VM's NIC.](images/Exercise3/waf-ipconfig.png "IP configuration link")

3. Set the **Public IP address** to **Associate**, and create a new public IP address named **UbuntuWAF-IP**. Choose a **Basic** tier IP address with **Dynamic** assignment. **Save** your changes.

    ![Screenshot showing the public IP configured on the UbuntuWAF VM.](images/Exercise3/waf-ip.png "Public IP configuration")

4. Return to the **UbuntuWAF** VM overview blade and copy the **Public IP address** value.

    ![Screenshot showing the IP address for the UbuntuWAF VM.](images/Exercise3/ubuntu-public-ip.png "UbuntuWAF public IP address")

5. Open a new browser tab and paste the IP address into the address bar. Verify that the SmartHotel360 application is now available in Azure.

    ![Screenshot showing the SmartHotel application.](images/Exercise3/smarthotel.png "Migrated SmartHotel application")

#### Task summary 

In this task, you assigned a public IP address to the UbuntuWAF VM and verified that the SmartHotel application is now working in Azure.

### Task 10: Post-migration steps

There are a number of post-migration steps that should be completed before the migrated services is ready for production use. These include:

- Installing the Azure VM Agent

- Cleaning up migration resources

- Enabling backup and disaster recovery

- Encrypting VM disks

- Ensuring the network is properly secured

- Ensuring proper subscription governance is in place, such as role-based access control and Azure Policy

- Reviewing recommendations from Azure Advisor and Security Center

In this task you will install the Azure Virtual Machine Agent (VM Agent) on your migrated Azure VMs and clean up any migration resources. The remaining steps are common for any Azure application, not just migrations, and are therefore out of scope for this hands-on lab.

> **Note**: The Microsoft Azure Virtual Machine Agent (VM Agent) is a secure, lightweight process that manages virtual machine (VM) interaction with the Azure Fabric Controller. The VM Agent has a primary role in enabling and executing Azure virtual machine extensions. VM Extensions enable post-deployment configuration of VM, such as installing and configuring software. VM extensions also enable recovery features such as resetting the administrative password of a VM. Without the Azure VM Agent, VM extensions cannot be used.
>
> In this lab, you will install the VM agent on the Azure VMs after migration. Alternatively, you could instead install the agent on the VMs in Hyper-V before migration.

1. In the Azure portal, locate the **smarthotelweb1** VM and open a remote desktop session using Azure Bastion. Log in to the **Administrator** account using password **demo!pass123** (use the 'eyeball' to check the password was entered correctly with your local keyboard mapping).

2. Open a web browser and download the VM Agent from:

    ```s
    https://go.microsoft.com/fwlink/?LinkID=394789
    ```

    **Note**: You may need to open the clipboard panel on the left-hand edge of the Bastion window, paste the URL, and then paste into the VM.

3. After the installer has downloaded, run it. Select **Next**, Select **I accept the terms in the License Agreement**, and then **Next** again. Select **Finish**.

    ![Screenshot showing the Windows installer for the Azure VM Agent.](images/Exercise3/vm-agent-win.png "VM Agent install - Windows")

4. Close the smarthotelweb1 window. Repeat the Azure VM agent installation process on **smarthotelweb2**.

You will now install the Linux version of the Azure VM Agent on the Ubuntu VM. All Linux distributions supports by Azure have integrated the Azure VM Agent into their software repositories, making installation easy in most cases.

5. In the Azure portal, locate the **UbuntuWAF** VM and **Connect** to the VM using Azure Bastion, with the user name **demouser** and password **demo!pass123**. Since this is a Linux VM, Bastion will create an SSH session. You may need to enter the credentials again. 
 
6. In the SSH session, enter the following command:

    ```s
    sudo apt-get install walinuxagent
    ```

    When prompted, enter the password **demo!pass123**. At the *Do you want to continue?* prompt, type **Y** and press **Enter**.

    **Note**: You may need to open the clipboard panel on the left-hand edge of the Bastion window, paste the command, and then paste into the VM.

    ![Screenshot showing the Azure VM Agent install experience on Ubuntu.](images/Exercise3/ubuntu-agent-1.png "VM agent install - Linux")

7. Wait for the installer to finish, then close the terminal window and the Ubuntu VM window.

To demonstrate that the VM Agent is installed, we will now execute the 'Run command' feature from the Azure portal. For more information on the VM Agent, see [Windows VM Agent](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows) and [Linux VM Agent](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux).

8. Navigate to the **smarthotelweb1** blade. Under **Operations**, select **Run command**, followed by **IPConfig**, followed by **Run**. After a few seconds, you should see the output of the IPConfig command.

    ![Screenshot showing the Run command feature.](images/Exercise3/run-command.png "Run command")

9. As a final step, you will now clean up the resources that were created to support the migration and are no longer needed. These include the Azure Migrate project, the Recovery Service Vault (Azure Site Recovery resource) used by  Azure Migrate: Server Migration, and the Database Migration Service instance. Also included are various secondary resources such as the Log Analytics workspace used by the Dependency Visualization, the storage account used by Azure Migrate: Server Migration, and a Key Vault instance.

    Because all of these temporary resources have been deployed to a separate **AzureMigrateRG** resource group, deleting them is as simple as deleting the resource group. Simply navigate to the resource group blade in the Azure portal, select **Delete resource group** and complete the confirmation prompts.

#### Task summary <!-- omit in toc -->

In this task you installed the Azure Virtual Machine Agent (VM Agent) on your migrated VMs. You also cleaned up the temporary resources created during the migration process.

### Exercise summary <!-- omit in toc -->

In this exercise you migrated the web tier and application tiers of the application from on-premises to Azure using Azure Migrate: Server Migration. Having migrated the virtual machines, you reconfigured the application tier to use the migrated application database hosted in Azure SQL Database, and verified that the migrated application is working end-to-end. You also installed the VM Agent on the migrated virtual machines, and cleaned up migration resources.

