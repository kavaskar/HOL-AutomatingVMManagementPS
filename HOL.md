<a name='HOLTop' />
# Managing Virtual Machines with the Windows Azure PowerShell Cmdlets #

<a name='Overview' />
## Overview ##
In this hands-on lab you will understand the capabilities of automating the deployment and management of virtual machines in Windows Azure.


<a name='Objectives' />
### Objectives ###
In this hands-on lab, you will learn how to:

- Provision Virtual Machines
- Post Provision Configuration 
- Remote Reboot or Start
- Manage Disk and Image Libraries
- Export and Import Virtual Machines

<a name='Prerequisites' />
### Prerequisites ###

The following is required to complete this hands-on lab:

- [Windows Azure PowerShell Cmdlets](http://msdn.microsoft.com/en-us/library/windowsazure/jj156055)
- A Windows Azure subscription with the Virtual Machines Preview enabled - you can sign up for free trial [here](http://bit.ly/WindowsAzureFreeTrial)

>**Note:** This lab was designed to use Windows 7 Operating System.

<a name="Setup"/>
### Setup ###

In order to execute the exercises in this hands-on lab you need to set up your environment.

1. Open a Windows Explorer window and browse to the lab’s **source** folder.

1. Execute the **Setup.cmd** file with Administrator privileges to launch the setup process that will configure your environment and install the Visual Studio code snippets for this lab.

1. If the User Account Control dialog is shown, confirm the action to proceed.

	>**Note:** Make sure you have checked all the dependencies for this lab before running the setup.

---

<a name='Exercises' />
## Exercises ##

This hands-on lab includes the following exercises:

1. [Provisioning a Virtual Machine using PowerShell CmdLets](#Exercise1)
1. [Using PowerShell CmdLets for Advanced Provisioning](#Exercise2)

<a name='gettingstarted' />
### Getting Started: Obtaining Subscription's Credentials ###

In order to complete this lab, you will need your subscription’s secure credentials. Windows Azure lets you download a Publish Settings file with all the information required to manage your account in your development environment.

#### Task 1 - Downloading and Importing a Publish-settings File ####

> **Note:** If you have done these steps in a previous lab on the same computer you can move on to Exercise 1.

In this task, you will log on to the Windows Azure Portal and download the publish-settings file. This file contains the secure credentials and additional information about your Windows Azure Subscription to use in your development environment. Then, you will import this file using the Windows Azure Cmdlets in order to install the certificate and obtain the account information.

1.	Open an Internet Explorer browser and go to <https://windows.azure.com/download/publishprofile.aspx>.

1.	Sign in using the credentials associated with your Windows Azure account.

1.	**Save** the publish-settings file to your local machine.

	![Downloading publish-settings file](images/downloading-publish-settings-file.png?raw=true 'Downloading publish-settings file')

	_Downloading publish-settings file_

	> **Note:** The download page shows you how to import the publish-settings file using Visual Studio Publish box. This lab will show you how to import it using the Windows Azure PowerShell Cmdlets instead.

1. In the start menu under **Windows Azure**, right-click **Windows Azure PowerShell** and choose **Run as Administrator**.

1.	Change the PowerShell execution policy to **RemoteSigned**. When asked to confirm press **Y** and then **Enter**.
	
	<!-- mark:1 -->
	````PowerShell
	Set-ExecutionPolicy RemoteSigned
	````

	> **Note:** The Set-ExecutionPolicy cmdlet enables you to determine which Windows PowerShell scripts (if any) will be allowed to run on your computer. Windows PowerShell has four different execution policies:
	>
	> - _Restricted_ - No scripts can be run. Windows PowerShell can be used only in interactive mode.
	> - _AllSigned_ - Only scripts signed by a trusted publisher can be run.
	> - _RemoteSigned_ - Downloaded scripts must be signed by a trusted publisher before they can be run.
	> - _Unrestricted_ - No restrictions; all Windows PowerShell scripts can be run.
	>
	> For more information about Execution Policies refer to this TechNet article: <http://technet.microsoft.com/en-us/library/ee176961.aspx>

	
1.	The following script imports your publish-settings file and generates an XML file with your account information. You will use these values during the lab to manage your Windows Azure Subscription. Replace the placeholder with your publish-setting file’s path and execute the script.

	<!-- mark:1 -->
	````PowerShell
	Import-AzurePublishSettingsFile '[YOUR-PUBLISH-SETTINGS-PATH]'   
	````

1. Execute the following commands and take note of the Subscription name and the storage account name you will use for the exercise.

	<!-- mark:1-3 -->
	````PowerShell
	Get-AzureSubscription | select SubscriptionName
	Get-AzureStorageAccount | select StorageAccountName 
	````

1. If you do NOT have a storage account returned above you should create one first.  
Run the following to determine the data center to create your storage account in. Ensure you pick a data center that shows support for PersistentVMRole. 


	````PowerShell
	Get-AzureLocation  
	````

1. Create your storage account: 


	````PowerShell
	New-AzureStorageAccount -StorageAccountName '[YOUR-SUBSCRIPTION-NAME]' -Location '[DC-LOCATION]'
	````

1. Execute the following command to set your current storage account for your subscription.


	````PowerShell
	Set-AzureSubscription -SubscriptionName '[YOUR-SUBSCRIPTION-NAME]' -CurrentStorageAccount '[YOUR-STORAGE-ACCOUNT]'
	````

<a name='Exercise1' />
### Exercise 1: Provisioning a Virtual Machine using PowerShell CmdLets###

In this exercise, you will learn how to provision a simple virtual machine in Windows Azure using PowerShell. 

<a name='Ex1Task2'></a>
#### Task 2 - Provisioning a Virtual Machine####

The first step to create a virtual machine in Windows Azure is to define the virtual machine configuration for items such as endpoints or data disks, and then define which cloud service and data center the vm will reside in. 

1. Execute the following command to retrieve the available data center locations. 

	````PowerShell
	Get-AzureLocation | select name
	````

1. Define the $dclocation variable with the name of the data center you want to deploy to.

	````PowerShell
	$dclocation = '[YOUR-LOCATION]'
	````
	Once the location is selected, you will need to create the virtual machine configuration. 

	To create virtual machines you will need a few pieces of information: The cloud service name that the VM will be contained in, and the VM image name.

1. Select a unique name for your cloud service. To validate the name is not in use you can use **Test-AzureName** cmdlet. It will return true if the service name already exists.

	````PowerShell
   Test-AzureName -Service '[YOUR-CLOUD-SERVICE-NAME]'

	$cloudSvcName = '[YOUR-CLOUD-SERVICE-NAME]'
	````

1. Select the VM Image you want to use as the basis of the VM.

	````PowerShell
   # Retrieves all available VM Images 
   Get-AzureVMImage | select ImageName
	````

	````PowerShell
	$image = '[YOUR-SELECTED-IMAGE-NAME]'
	````

1. Next, choose the VM creation script below based on whether you selected Windows or Linux. 

	1. A Windows Windows Virtual Machine from an Image.

		````PowerShell
		$adminPassword = '[YOUR-PASSWORD]'
		$vmname = 'mytestvm1'
			
		New-AzureQuickVM -Windows -ServiceName $cloudSvcName -Name $vmname -ImageName $image -Password $adminPassword -Location $dclocation
		````

	1. A Linux Virtual Machine from an Image. Notice that the image has changed, and the -OS switch specifies Linux as the operating system.

		````PowerShell
		$linuxuser = '[CHOOSE-USERNAME]'
		$adminPassword = '[YOUR-PASSWORD]'
		$vmname = 'mytestvm1'
			
		New-AzureQuickVM -Linux -ServiceName $cloudSvcName -Name $vmname -ImageName $image -LinuxUser $linuxuser -Password $adminPassword -Location $dclocation
		````

	>**Note:** Specifying -Location on New-AzureQuickVM or New-AzureVM tells the cmdlet to attempt to create a cloud service as a container for the virtual machines. Use this option when creating the first virtual machine and omit it when adding new virtual machines to the same cloud service.


1. Once the virtual machine has been created you can inspect it using the **Get-AzureVM** cmdlet. The following command will enumerate the details of all the virtual machines in the cloud service.

	````PowerShell
	Get-AzureVM -ServiceName $cloudSvcName 
	````

	To be more specific you can use the -Name parameter.

	````PowerShell
	Get-AzureVM -ServiceName $cloudSvcName -Name $vmname
	````

1. The **Windows Azure PowerShell Cmdlets** support restart, stop and start operations as well using the **Restart-AzureVM**, **Stop-AzureVM** and **Start-AzureVM** commands. 

	With the following commands you will be able to start, stop and restart your VM.

	````PowerShell
	# Restart
	Restart-AzureVM -ServiceName $cloudSvcName -Name $vmname

	# Shutdown 
	Stop-AzureVM -ServiceName $cloudSvcName -Name $vmname

	# Start
	Start-AzureVM -ServiceName $cloudSvcName -Name $vmname
	````

	>**Note:** Make sure your virtual machine finished provisioning before executing these commands.

<a name='Exercise2'/>
### Exercise 2: Advanced Provisioning Settings##

In addition to just creating a single uncustomized virtual machine. You can also configure data disks, disk cache settings, networking endpoints and automatically configure domain join settings at provisioning time in addition to batch creating VMs using the New-AzureVMConfig/New-AzureVM cmdlet combination.

The example below creates two new virtual machines with a 50 GB data disk already attached and a load balanced endpoint open on port 80 for http traffic.

>**Note:** You will still need to log into the machine and configure/format the data disk via disk manager. In the next section you will find a walk through for these steps.

Example: Creating multiple VMs (Windows)

````PowerShell
   $vmname2 = 'mytestvm2'
   $vmname3 = 'mytestvm3'


   $vm2 = New-AzureVMConfig -Name $vmname2 -InstanceSize ExtraSmall -ImageName $image |
		     Add-AzureProvisioningConfig -Windows -Password $adminPassword |
		     Add-AzureDataDisk -CreateNew -DiskSizeInGB 50 -DiskLabel 'datadisk1' -LUN 0 |
		     Add-AzureEndpoint -Protocol tcp -LocalPort 80 -PublicPort 80 -Name 'web' `
				 -LBSetName 'lbweb' -ProbePort 80 -ProbeProtocol http -ProbePath '/' 
	
   $vm3 = New-AzureVMConfig -Name $vmname3 -InstanceSize ExtraSmall -ImageName $image |
          Add-AzureProvisioningConfig -Windows -Password $adminPassword  |
		   Add-AzureDataDisk -CreateNew -DiskSizeInGB 50 -DiskLabel 'datadisk2' -LUN 0  |
		   Add-AzureEndpoint -Protocol tcp -LocalPort 80 -PublicPort 80 -Name 'web' `
				 -LBSetName 'lbweb' -ProbePort 80 -ProbeProtocol http -ProbePath '/' 

   New-AzureVM -ServiceName $cloudSvcName -VMs $vm2,$vm3
````

Example: Creating a second VM (Linux)

````PowerShell
   $vmname2 = 'mytestvm2'
   $vmname3 = 'mytestvm3'

   $vm2 = New-AzureVMConfig -Name $vmname2 -InstanceSize ExtraSmall -ImageName $image |
          Add-AzureProvisioningConfig -Linux -LinuxUser $linuxUser -Password $adminPassword |
          Add-AzureDataDisk -CreateNew -DiskSizeInGB 50 -DiskLabel 'datadisk1' -LUN 0 |
		   Add-AzureEndpoint -Protocol tcp -LocalPort 80 -PublicPort 80 -Name 'web' `
				 -LBSetName 'lbweb' -ProbePort 80 -ProbeProtocol http -ProbePath '/' 
	
   $vm3 = New-AzureVMConfig -Name $vmname3 -InstanceSize ExtraSmall -ImageName $image |
			Add-AzureProvisioningConfig -Linux -LinuxUser $linuxUser -Password $adminPassword |
			Add-AzureDataDisk -CreateNew -DiskSizeInGB 50 -DiskLabel 'datadisk2' -LUN 0 |
			Add-AzureEndpoint -Protocol tcp -LocalPort 80 -PublicPort 80 -Name 'web' `
				-LBSetName 'lbweb' -ProbePort 80 -ProbeProtocol http -ProbePath '/' 

   New-AzureVM -ServiceName $cloudSvcName -VMs $vm2,$vm3
````

<a name='Ex2Task1'></a>
#### Task 1 - Post Provisioning Configuration ####

Modifying an existing virtual machine requires retrieving the current settings by calling **Get-AzureVM**, modifying them and then calling the **Update-AzureVM** cmdlet to save the changes.

You can hot add and remove data disks and networking endpoints. Changing disk cache settings requires a reboot as does changing the virtual machine's instance size. 

The following example uses the **Get-AzureVM** cmdlet to retrieve the VM object and send it to the PowerShell Pipeline.

**Add-AzureDataDisk** with the **CreateNew** parameter allows you to dynamically add storage to the virtual machine. In this case we are calling it twice to attach to unformatted blank VHDs to the server each 50 gigs of storage each. The -LUN parameter tells the order of the device being attached and optionally uses the -MediaLocation to specify the location in Storage to keep the newly created VHDs.

**Add-AzureDataDisk** also supports the **Import** parameter to attach a disk in the disk library and **-ImportFrom** to attach a disk that already exists in storage. 

The example also adds a new endpoint for TCP port 1433 internally that is listening externally on port 2000 using the **Add-AzureEndpoint** command. 

>**Note:** to connect to SQL you would still need to enable 1433 on the Windows Server firewall to connect

1. Use the following script to Hot Add data disks and endpoints to an existing virtual machine. 

	````PowerShell
	$vmname = 'mytestvm1'

	$vm = Get-AzureVM -Name $vmname -ServiceName $cloudSvcName |
		Add-AzureDataDisk -CreateNew -DiskSizeInGB 50 -DiskLabel 'datadisk1' -LUN 0 |
		Add-AzureDataDisk -CreateNew -DiskSizeInGB 50 -DiskLabel 'translogs1' -LUN 1 |
		Add-AzureEndpoint -Protocol tcp -LocalPort 1433 -PublicPort 2000 -Name 'sql' |
		Update-AzureVM 
	````

1. Once the **Update-AzureVM** call has completed you will need to log into the machine to complete configuring the disks or have remote management enabled to complete the disk initialization via PowerShell.

	![uninitdisk](images/uninitdisk.png?raw=true)

	_VM Disks_

1. Right-click on each disk (on the left side) and mark it online. 

1. Once the disks are online you will need to right-click on one and click **Initialize** (on the left side).

	![InitializeDisk](images/initializedisk.png?raw=true)

	_Initializing disks_

1. Once the disks are initialized you will then need to right click on the right side and select Create Simple Volume (software RAID is support so those are options are available as well). The create simple volume wizard will allow you to format the disks and mount them for use.

	![formatteddisks](images/formatteddisks.png?raw=true)

	_Formatted disks_

1. You can control the disk cache settings for your data disks by calling **Set-AzureDataDisk** and configuring the HostCaching parameter. Valid values for HostCaching are _ReadOnly_, _ReadWrite_ and _None_.

	>**Note:** This change is at the host level and will not be reflected in disk manager. 
	> By default write cache is disabled and read cache is enabled on data disks.

1. With the following script you are enabling Write Cache on a data disk and viewing the resulting change.

	````PowerShell
	$vm = Get-AzureVM -Name $vmName -ServiceName $cloudSvcName  |
		   Set-AzureDataDisk -HostCaching ReadWrite -LUN 0 |
		   Set-AzureDataDisk -HostCaching ReadWrite -LUN 1 |
		   Update-AzureVM

	Get-AzureVM -ServiceName $cloudSvcName -Name $vmname | Get-AzureDataDisk
	````

<a name='Ex2Task2' />

#### Task 2 - Changes that Require a Reboot ####


Some changes require the virtual machine to be **restarted** when applied. Making changes to the underlying hardware by changing the instance size using Set-AzureRoleSize, modifying the OS Disk cache settings with Set-AzureOSDisk or moving the virtual machine between subnets using Set-Subnet all will result in an automatic restart of the virtual machine.

1. Run the following script to disable the write disk cache, changing the write cache setting of the OS disk from _write cache enabled_ to _write cache disabled_. Once executed the virtual machine will restart with the new settings. 

	````PowerShell
	Get-AzureVM -ServiceName $cloudSvcName -Name $vmName |
		Set-AzureDataDisk -HostCaching ReadOnly -LUN 0|
		Set-AzureDataDisk -HostCaching ReadOnly -LUN 1|
		Update-AzureVM 
	````

1. Run the following script to change the instance size of a Virtual Machine.

	>**Note:** The snippet below sets the instance size of the specified virtual machine. This does require a reboot as the new hardware is provisioned. 
	
	````PowerShell
	Get-AzureVM -ServiceName $cloudSvcName -Name $vmName |
		Set-AzureVMSize -InstanceSize Medium |
		Update-AzureVM
	````

<a name='Ex2Task3'></a>
#### Task 3 - Managing disk images ####

It is simple to view all of the data disks or images in the disk and image repository with PowerShell.
Running the Get-AzureDisk command will enumerate all of the data disks in your subscription.

1. Use the following command to retrieve all your subscription's disks.
	
	````PowerShell
	Get-AzureDisk
	````

1. You can use PowerShell's built in capabilities to limit the results. For instance, with this example you will be able to find a specific virtual machine's VHD image.

	````PowerShell
	$vmname = 'mytestvm2'
	Get-AzureDisk | Where { $_.AttachedTo.RoleName -eq $vmname }
	````

1. Currently, when a virtual machine is removed the underlying VHDs are not removed as well. PowerShell allows you to clean up the underlying storage when removing a virtual machine.

	The following script removes a specific Virtual Machine as well as its disks.

	````PowerShell
	$vmname = 'mytestvm2'
	$vmDisks = Get-AzureDisk | Where { $_.AttachedTo.RoleName -eq $vmname }

	Remove-AzureVM -ServiceName $cloudSvcName -Name $vmname 

	$vmDisks | foreach {
		Remove-AzureDisk -DiskName $_.DiskName -DeleteVHD
	}
	````



1. Similar functionality exists for managing the image repository on your subscription. With this script you will Identify User Created Images.

	````PowerShell
	Get-AzureVMImage | Where { $_.Category -eq 'User' }
	````

<a name='Ex2Task4' />
####Task 4 - Imaging, Exporting and Importing Virtual Machine Configurations ####

Windows Azure IaaS provides the capability to customize a virtual machine, generalize it using a tool like sysprep, and then capture the virtual machine to the image library. This functionality allows you to create customized images that you can then re-use to generate multiple identical machines. The steps to accomplish this from PowerShell are relatively simple.

1. Execute the following script to create a VM that will be the Start of the Image.

	1. For Windows VMs.

		````PowerShell
	   $vmname = 'winvmforimg'
		New-AzureVMConfig -Name $vmname -InstanceSize Small -ImageName $image |
			Add-AzureProvisioningConfig -Windows -Password $adminPassword |
			New-AzureVM -ServiceName $cloudSvcName 
		````

	1. For Linux VMs.

		````PowerShell
	   $vmname = 'linuxvmforimg'
		New-AzureVMConfig -Name $vmname -InstanceSize Small -ImageName $image |
			Add-AzureProvisioningConfig -Linux -LinuxUser $linuxuser -Password $adminPassword |
			New-AzureVM -ServiceName $cloudSvcName 
		````

1. Generalizing a Virtual Machine for Capture. At this point you would customize the VM with settings required for the captured image. 

	1. Connect to the Virtual Machine using either RDP or SSH 

	1. For Windows, sysprep from within Windows. Select Entire System Out-of-Box Experience (OOBE), check **Generalize** and select **Shutdown**.

		![sysprep](images/sysprep.png?raw=true)

		_SysPrep_

	1. For Linux Virtual Machines, run the following script.

		````PowerShell
		sudo /user/sbin/waagent -deprovision+user
		````

1. Generate a new image using the **Save-AzureVMImage** cmdlet. 

	>**Note:** The VM must be completely shut down before running the Save-AzureVMImage cmdlet. You can check the status of the VM by typing in Get-AzureVM -Name vmname.

	````PowerShell
	Save-AzureVMImage -ServiceName $cloudSvcName -Name $vmname -NewImageName [YOUR-NEW-VM-IMAGE-NAME] -NewImageLabel [YOUR-NEW-IMAGE-LABEL] -PostCaptureAction Delete
	````

	> **Note:** The **Save-AzureVMImage** cmdlet makes a running persistent VM available as an image for reuse. For Windows VMs, the image should be sysprepped before capture. After performing the capture, you can delete or reprovision the VM using the PostCaptureAction parameter with Delete|Reprovision value.


<a name='Ex2Task5' />
#### Task 5 - Exporting and Importing VM configuration ####

The Windows Azure PowerShell Cmdlets provide the capability of saving the configuration of a virtual machine. 
This is useful in scenarios where you need to completely remove the virtual machine but at some point (easily) put it back. It works by understanding the fact that when you remove a virtual machine by default the underlying data and OS disk in storage is not removed. The Export-AzureVM cmdlet saves all of the configuration of the virtual machine including the disk names, endpoint settings etc.. to an XML file. This allows you to delete the virtual machine and then later re-create it using the saved configuration.


1. Run the following script to export the VM Configuration and remove the Deployment.

	````PowerShell
	$vmname = 'mytestvm1' 
	Export-AzureVM -ServiceName $cloudSvcName -Name $vmname -Path 'c:\Temp\mytestvm1-config.xml' 
	Remove-AzureVM -ServiceName $cloudSvcName -Name $vmname
	````

	>**Note:** This code saves the configuration of the mytestvm1 virtual machine and then removes it by removing the vm.
	>
	>Make sure you created a Temp folder within C: drive before executing the Command.

1. Once the deployment has been removed you can then recreate the virtual machine from the saved state. Run the following script to import the VM Configuration into a New Deployment.

	````PowerShell
	Import-AzureVM -Path 'c:\Temp\mytestvm1-config.xml' | 
		New-AzureVM -ServiceName $cloudSvcName 
	````

	>**Note:** In deployments with Virtual Networking this will result in a new IP address so it is not recommended for VMs that require a persistent IP such as a domain controller.

<a name='Ex2Task6' />
#### Task 6 - Managing RDP and SSH Connectivity ####

By default all new virtual machines created from the Windows Azure PowerShell cmdlets will allow RDP for Windows or SSH. To disable automatic endpoint creation use the **DisableRDP** or **DisableSSH** arguments. This tells the cmdlets not to create the RDP or SSH endpoints during provisioning.

To discover the ports for these endpoints you can use the Get-AzureEndpoint to learn the public port of the RDP or SSH input endpoint.

1. Endpoint Discovery.

	````PowerShell
	Get-AzureVM -ServiceName $cloudSvcName -Name $vmname | Get-AzureEndpoint
	````

1. Save the RDP File to the filesystem.

	````PowerShell
	Get-AzureRemoteDesktopFile -ServiceName $cloudSvcName -Name $vmname -LocalPath C:\Temp\myvmconnection.rdp
	````

1. Launch the RDP Client directly after downloading the RDF File.

	You can use the Windows Azure PowerShell cmdlets to save the RDP file for Windows Machines or launch it directly from PowerShell.

	````PowerShell
	Get-AzureRemoteDesktopFile -ServiceName $cloudSvcName -Name $vmname -LocalPath C:\Temp\myvmconnection-2.rdp -Launch 
	````

---

<a name='Summary' />
## Summary ##

In this hands-on lab you were shown how to configure your subscription id and certificate to manage Windows Azure Virtual Machines. You were also shown the basics of how to provision virtual machines and modify them with hot add capabilities and changes that require reboots such as changing the instance size. 

In addition you were shown how you can use the Windows Azure PowerShell cmdlets to manage your disk and image libraries along with the capability of exporting and import virtual machine configurations.