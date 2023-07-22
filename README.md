<h1>SIEM in Microsoft Sentinal (Azure)</h1>

 ### [YouTube Demonstration](https://youtu.be/7eJexJVCqJo)

<h2>Description</h2>
Project consists of a simple PowerShell script that walks the user through "zeroing out" (wiping) any drives that are connected to the system. The utility allows you to select the target disk and choose the number of passes that are performed. The PowerShell script will configure a diskpart script file based on the user's selections and then launch Diskpart to perform the disk sanitization.
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Diskpart</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Walkthrough</h2>

<h3>Before Starting</h3>
The first thing that I did was set up an Azure account. I had to enter my credit card information but Microsoft provides new users with $200 of free Azure credits that they can use over a month, which is enough for this lab. 

<h3>After Account Setup</h3>
Next, I head on to https://portal.azure.com:
<img src="https://i.imgur.com/52ILMGM.png" height="80%" width="80%" alt="Azure Portal"/> 


<h4>Creating a Virtual Machine</h4>
This is the machine that is going to be exposed on the internet. Different people from all around the world will attempt to attack this machine and try to login to this machine. Essentially, This machine acts as a <b>Honeypot</b> for this lab. 
<br />
<br />

Search "Virtual Machine" on the search bar and click "Virtual Machines" under "Services"
<br/>
<img src="https://i.imgur.com/06SyeQK.png" height="80%" width="80%" alt="Select Virtual Machines"/>
<br />
<img src="https://i.imgur.com/0UCxxwh.png" height="80%" width="80%" alt="Virtual Machines Page"/>
<br />
Create a new Azure Virtual Machine:<br />
<img src="https://i.imgur.com/oEE75SH.png" height="80%" width="80%" alt="Create a New Virtual Machine Button"/>
<br />
Create a new Resource Group:  <br/>
<img src="https://i.imgur.com/dHDRpPb.png" height="80%" width="80%" alt="Create a New Resource Group"/>
<br />
A resource group in Azure is a logical grouping of resources that usually share the same lifespan. Everything in this lab project are going to be in the same resource group. 

I changed the following information on the "Create Virtual Machine" page for the lab:<br />
<b>Resource Group:</b> Create New >> "HoneyPotLab"<br />
<b>Virtual Machine Name:</b> "HoneyPot-VirtualMachine"<br />
<b>Region:</b> "(US) East US" -> <i>This is the location of the data center that will be used for this Virtual Machine</i><br />
<b>Image:</b> "Windows 10 Pro, version 22H2 - x64 Gen2"<br />
<b>Size:</b> "Standard_B1s - 1 vcpu, 1GiB memory"<br /><br />

<b>Username:</b> "cybersuhanlab"<br />
<b>Password: </b> "<i>mypassword</i>"<br />
<b>Confirm Password: </b><i>Repeat Password</i><br />

<i>I will be using the above username and password to log in to the virtual machine later.<i/><br /><br />

<b>Public Inbound Ports:</b> "Allow selected ports" -> <i>Default</i> <br />
<b>Selected Inbound ports:</b> " RDP(3389) -> <i>Default</i><br />
<b><i>Tick the Licencing rights note</i></b><br /><br />
<img src="https://i.imgur.com/a2ZdWqQ.png" height="80%" width="80%" alt="VM Settings 1"/><br />
<img src="https://i.imgur.com/bGMlut3.png" height="80%" width="80%" alt="VM Settings 2"/><br />
<br />
Next, I go to Disks settings for the Virtual Machine by clicking on "Next: Diks >"
<img src="https://i.imgur.com/Y1MQvA6.png" height="80%" width="80%" alt="Disk Settings"/><br />
I leave the Disk settings as default for this lab and Click on "Next: Networking >"<br />

Create New Network Security Group: <br />

At this point, I want my firewall to be open to everything to the public internet. For this, I have to create a firewall inbound rule that allows everything into the VM. For this, I create a new network security group and set open inbound rules. The point here is that I want to make my lab machine very discoverable by any means necessary. I dont want my VM to drop any traffic. To do this, I take the following steps:

In the Networking settings, I create a new Network Security Group: <br />
<img src="https://i.imgur.com/JIxAbvl.png" height="80%" width="80%" alt="VM Networking Page"/><br />

In the network security group page, I remove the existing inbound firewall rules: <br />
<img src="https://i.imgur.com/9c4DXnW.png" height="80%" width="80%" alt="Remove Inbound Rules"/><br />

Then, I add a new inbound rule with the following configuration: <br />

<b>Source: </b> "Any"<br />
<b>Source Port Ranges: </b> "*"<br />
<b>Destination: </b>"Any"<br />
<b>Service: </b>"Custom"<br />
<b>Destination port ranges: </b>"*"<br />
<b>Protocol: </b>"Any"<br />
<b>Action: </b>"Allow"<br />
<b>Priority: </b>"100" -> <i>Set to lower priority</i><br />
<b>Name: </b>"ANY_INBOUND_CAUTION"<br />

Then I click Add to add the inbound rule.<br />
<img src="https://i.imgur.com/tAIsWvn.png" height="80%" width="80%" alt="Add Inbound Rule"/><br />
<img src="https://i.imgur.com/IcKkLDF.png" height="80%" width="80%" alt="Update Inbound Rule"/><br />
<img src="https://i.imgur.com/mE2xi5w.png" height="80%" width="80%" alt="Confirm Rule"/><br />

I click OK to save my new network configuration group.<br /><br />

Then for the last step of creating a virtual machine, I click "Review and Create"<br />
<img src="https://i.imgur.com/B6rEVgK.png" height="80%" width="80%" alt="Review and Create"/><br />

Finally, I get to the Review Page where I press "Create" to Deploy my new Virtual Machine<br />
<img src="https://i.imgur.com/RIiz3sq.png" height="80%" width="80%" alt="Create VM"/><br />

<h4>Creating a new Log Analytics Workspace</h4>

The purpose of a Log Analytics Workspace is to inject logs from the Virtual Machine, specifically the Windows event logs. I will also create a custom log that contains geographic information so that I can discover where the attacks are coming from. The log analytics workspaces are where the logs are stored. Then, Azure Sentinel will connect to this workspace to display the geographical data on the map. 

To create a log analytics workspace, I take the following steps: <br />



<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
