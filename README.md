<h1>SIEM (Microsoft Sentinel) in Azure</h1>

 ### [YouTube Demonstration](https://YT_LINK_HERE)

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

To create a log analytics workspace, I take the following steps: <br /><br />

Open Log Analytics Page: <br />
<img src="https://i.imgur.com/9j3WXa2.png" height="80%" width="80%" alt="Search Log Analytics"/><br />

Click on Create: <br />
<img src="https://i.imgur.com/F1dDRDk.png" height="80%" width="80%" alt="Create Log Analytics"/><br />

In the Create Log Analytics Workspace page, I select the <b>"HoneyPotLab"</b> as the resource group as it is the one I created in the previous step. For the Instance name, I name it <b>"LAWHoneyPot"</b> as Log Analytics Workspace HoneyPot. Then I press "Review + Create". On the next page, I review the configuration I set and press create. <br />

<img src="https://i.imgur.com/gSgyLJP.png" height="80%" width="80%" alt="Log Analytics Config"/><br />
<img src="https://i.imgur.com/UgbNaIs.png" height="80%" width="80%" alt="Review and Create Log Analytics"/><br /><br />

Now, I need to enable the ability to gather log information from the Virtual Machine into the Log Analytics Workspace. For this, I go to the Security Center.<br />
<img src="https://i.imgur.com/aG5rQfU.png" height="80%" width="80%" alt="Review and Create Log Analytics"/><br /><br />

In the Security Center, I find "Environment Settings" on the left-hand panel and click on it.<br />
<img src="https://i.imgur.com/MLD95mu.png" height="80%" width="80%" alt="Click Environment Settings"/><br /><br />

Next, I click the dropdown of my Azure Subscription and find my Log Analytics Workspace (LAW). I click on Edit settings for my LAW:<br />
<img src="https://i.imgur.com/lC36iJP.png" height="80%" width="80%" alt="Edit LAW settings"/><br /><br />

I toggle the "SQL Server on Machines" to "Off" and Save the configuration:<br /> 
<img src="https://i.imgur.com/93Q3hfW.png" height="80%" width="80%" alt="Edit LAW settings"/><br /><br />

Next, I click on "Data Collection" on the left-hand panel. I select "All Events" from the list and save the configuration:<br />
<img src="https://i.imgur.com/8Qyugqn.png" height="80%" width="80%" alt="Data Collection"/><br /><br />

I need to connect the Log Analytics Workspace to the Virtual Machine. To do this I go back to the Log Analytics Workspace Page and click on my LAW:<br />
<img src="https://i.imgur.com/OpseDSA.png" height="80%" width="80%" alt="Data Collection"/><br /><br />

Then I find "Azure Virtual Machine" under "Connect Data Source" or "Virtual Machines" on the left-hand side panel:<br />
<img src="https://i.imgur.com/SvNChjo.png" height="80%" width="80%" alt="Connect Data Source"/><br /><br />

I find my Virtual Machine created in the first step and click on it. <br />
<img src="https://i.imgur.com/Xz4UU4F.png" height="80%" width="80%" alt="Select VM"/><br /><br />

On the next page, I press "Connect" to connect my Virtual Machine to my LAW:<br />
<img src="https://i.imgur.com/UnsFsnj.png" height="80%" width="80%" alt="Connect VM"/><br /><br />


<h4>Setting up Microsoft Sentinal</h4>
Microsoft Sentinal is the SIEM that I will use to visualize the attack data. <br /><br />

To Set Up Microsoft Sentinal, I search Sentinal on the Azure Portal Homepage and Click on it.<br />
<img src="https://i.imgur.com/NHU3cuw.png" height="80%" width="80%" alt="Search Sentinal"/><br /><br />

Next, I click on "Create Microsoft Sentinal."<br />
<img src="https://i.imgur.com/aRHbV3Q.png" height="80%" width="80%" alt="Search Sentinal"/><br /><br />

Then, I select the LAW that I created in the previous step and add it.<br />
<img src="https://i.imgur.com/LyOPwux.png" height="80%" width="80%" alt="Search Sentinal"/><br /><br />

At this point, Microsoft Sentinel is connected to our Log Analytics Workspace. 

<h4>Starting Azure Virtual Machine through Remote Connection</h4>
After I completed connecting Sentinel to the LAW, Now I establish a Remote Connection from my Machine to the Virtual Machine that I created in Azure. For this, I need the IP address of the Virtual Machine. To get the IP Address, I go to my Azure Portal and find "Virtual Machines" and click on it. The portal takes me to the list of my virtual machine where I select my HoneyPotLab Virtual Machine:<br /> 
<img src="https://i.imgur.com/qWEY87F.png" height="80%" width="80%" alt="Find Virtual Machine"/><br /><br />
<img src="https://i.imgur.com/IGpoNQN.png" height="80%" width="80%" alt="Select Virtual Machine"/><br /><br />

Next, I can see the <b>Public IP Address</b> of my Virtual Machine:<br />
<img src="https://i.imgur.com/88YComR.png" height="80%" width="80%" alt="VM IP Address"/><br /><br />

I copy the Public IP Address and on my device, I open <b>Remote Desktop Connection</b>. I paste the VM IP address and click "Connect". In the next window, I type the username and password that I set while configuring the VM and click "OK". If the credentials are correct, I get logged in to the VM in the Remote Desktop Connection Window. 
<img src="https://i.imgur.com/aeBAZxp.png" height="80%" width="80%" alt="RDC Window"/><br /><br />
<img src="https://i.imgur.com/SiVFr2r.png" height="80%" width="80%" alt="RDC Window Credentials"/><br /><br />

The Virtual Machine IP is shown on the top of the Remote Desktop Connection Window.
<img src="https://i.imgur.com/Tuf7IkD.png" height="80%" width="80%" alt="VM IP Address in RDC"/><br /><br />

<h4>Update Firewall settings in the Virtual Machine</h4>

For the sake of this lab, I want to make the virtual machine discoverable so that anyone over the internet can find the device and try to brute force and log in to the machine. For that, I need to update my firewall settings. I have two choices at this point. I can either allow ICMP inbound or turn the firewall off. For this lab, I choose to turn my firewall off and try to ping the virtual machine from my computer and I get a successful ping. <br />

Toggling VM Firewall Domain Profile off<br />
<img src="https://i.imgur.com/20b6W8r.png" height="80%" width="80%" alt="Firewall Domain Off"/><br /><br />
Toggling VM Firewall Private Profile off<br />
<img src="https://i.imgur.com/DqbqSRW.png" height="80%" width="80%" alt="Firewall Private Off"/><br /><br />
Toggling VM Firewall Public Profile off<br />
<img src="https://i.imgur.com/UZqhNbO.png" height="80%" width="80%" alt="Firewall Public Off"/><br /><br />
Pinging the VM from my computer<br />
<img src="https://i.imgur.com/yTRyLt6.png" height="80%" width="80%" alt="Pinging the VM"/><br /><br />

<h4>Custom Log Exporter</h4>

Next, I want a security log exporter made Powershell that will loop through the Event Log and grabs all the events in which users fail to log in, and grabs the IP Address. With the IP Address, I can get geolocation ta from ipgeolocation.io. The Exporter program exports failed login attempts from the event viewer to a file in "C:\ProgramData" in the Virtual Machine. The following screenshot is what the file looks like after 2 minutes of running the script:

<img src="https://i.imgur.com/2CWLMpM.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

<h4>Custom Log in Log Analytics Workspace</h4>

In my Azure Portal, I go to my Log Analytics Workspace (LAWHoneyPot) and Select Tables:<br />
<img src="https://i.imgur.com/rsZTZ8H.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

I create a New Custom Log Table (MMA Based):<br />
<img src="https://i.imgur.com/3FhrFy1.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

Then I copy the log file that the script output in previously and upload it as a sample log file: <br />
<img src="https://i.imgur.com/f3GVYG9.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

Next, I check the if the file is correct and the delimitation is right and I click <i>Next</i><br />
<img src="https://i.imgur.com/VXUTWLb.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

For the custom log name, I type in "FAILED_RDP_GEO" and click next. In Azure, the custom table can be queried as "FAILED_RDP_GEO_CL"<br />
<img src="https://i.imgur.com/GxkEZNc.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

In the VM, the custom log file is saved in "C:\ProgramData\failed_rdp.log", which is what I will write in the collection path. <br />
<img src="https://i.imgur.com/0pvt6TL.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

Next, I wait for about an hour before going to the Azure portal to my Log Analytics Workspace and click "Logs". I run the FAILED_RDP_GEO_CL query and get the data from the log file: <br />
<img src="https://i.imgur.com/zfaW6aF.png" height="80%" width="80%" alt="Custom Log"/><br /><br />

<b>At this point, I have the geographical data coming into the Log Analytics workspace as raw data in a custom table.</b>

<h4>Setting up a map in Microsoft Sentinal</h4>

Now that I am getting data in real-time, I want to set up Sentinal and plot the failed login attempts' location on a map. To do that, I take the following steps:<br /><br />

I go to Azure Portal and into Microsoft Sentinal. There, I click "Workbooks" from the left-hand panel and then click "Add Workbook"<br />

<img src="https://i.imgur.com/6NdoM9i.png" height="80%" width="80%" alt="Sentinal Workbooks"/><br /><br />

I add a query in the workbook and edit the query to <br />
<img src="https://i.imgur.com/9Gj87su.png" height="80%" width="80%" alt="Sentinal Workbooks"/><br /><br />
<img src="https://i.imgur.com/BLHv7B7.png" height="80%" width="80%" alt="Sentinal Workbooks"/><br /><br />







<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
