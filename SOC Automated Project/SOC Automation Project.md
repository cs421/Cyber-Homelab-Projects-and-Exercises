**Goal:** The goal of this project is to create an automated SOC analyst lab complete with the SIEM tool, as the case management tool, and a security automation platform for SOAR (Security Orchestration, Automation and Response) capabilities. This lab will also utilize **Mimikatz** as the sample credential harvesting attack.

### Tools to be used
1. **Wazuh** (https://wazuh.com) - XDR and SIEM tool, for receiving events and sending alerts to SOAR
2. **Shuffle** (https://shuffler.io) - for SOAR and IOC (indicators of compromise) enrichment
3. **TheHive** (https://thehive-project.org) - Security Incident Response Platform, for ticket and alert assignments to SOC analysts
4. **VirusTotal** (https://www.virustotal.com) - for IOC enrichment and threat intel
5. **Mimikatz** - (https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919) for sample credential harvesting attack


There will be **two (2)** virtual machines to be used in this project, namely:
1. A Windows 10 Client which will serve as the Wazuh agent, to be configured via VirtualBox.
2. An Ubuntu machine which will serve as another Wazuh agent later on, to be configured on the cloud.

**Wazuh, TheHive, and Shuffle** will be configured on the cloud.

I will be following **MyDFIR's** videos in creating this lab, as found in: https://www.youtube.com/watch?v=Lb_ukgtYK_U

# Part 1: Diagram

![[SOC Automated Homelab Diagram.png]]

**1.** The  **Wazuh** agent (Windows 10 client) will send events to the internet.
**2.** The **Wazuh** manager will then receive those events sent by the client and will create alerts.
**3.** **Wazuh** will send the alerts to **Shuffle**
**4.** **Shuffle** will send the alerts over to the Internet for IOC enrichment via OSINT (Open Source Intelligence), then back to **Shuffle**
**5.** **Shuffle** will then send the alerts over to **TheHive** for case management.
**6.** **Shuffle** will also send and email to the SOC analyst regarding the alerts.
**7.** The SOC analyst will receive the email sent by **Shuffle**. The email should contain details of the alert, and will ask the question, "Do you want to contain this event or not?" to the SOC analyst.
**8.** The SOC analyst will send the appropriate response action to **Shuffle**, which, in turn, will send the response back to the **Wazuh**.
**9.** **Wazuh** will instruct the agent to perform the appropriate responsive action.

# Part 2: Installing Virtual Machines and Applications

### Installing VirtualBox and Windows 10 virtual machine

Installing VirtualBox and the Windows 10 VM is very linear, as shown in MyDFIR's tutorial: https://www.youtube.com/watch?v=YxpUx0czgx4 from **2:07 - 8:34**

### Installing Sysmon
By now, the Windows 10 VM should have been installed and prepped. Open up a browser and go to https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon and choose **Download Sysmon**

![[download sysmon.png]]

### Downloading Sysmon configuration file

Go to https://github.com/olafhartong/sysmon-modular, scroll down and click **sysmonconfig.xml**
![[sysmon config.png]]
click **Raw**, then right click and **Save as** the xml file
![[sysmon config raw.png]]
![[sysmon xml save.png]]

Go to the Sysmon's download location, right click the zip file and select "Extract All"
![[sysmon extract.png]]

Move or drag and drop the sysmon config file into the sysmon folder.
![[sysmonconfig move.png]]

We will be needing **Powershell** in this next step. Go to start, and search for powershell, and select "Run as Administrator"

![[powershell admin.png]]
`cd` to the extracted sysmon folder
![[sysmon folder.png]]
**Note:** For good practice, always put paths inside quotation marks (" ")

We will now install sysmon along with the downloaded configuration file.

Run `.\ sysmon64.exe -i sysmonconfig.xml`
![[sysmon64 install.png]]

The license agreement window should appear. Click "Agree", and let the installer run its course
![[sysmon64 license agreement.png]]
![[sysmon install progress.png]]

To double check if Sysmon has been successfully installed, we can do the following steps:
##### Services
Open the **Services** app by typing 'services' in the start menu
![[services.png]]
Scroll down until you find **Sysmon64** in the list.
![[services sysmon.png]]

##### Event Viewer
Open the **Event Viewer** by typing 'event' in the start menu and clicking on the app
![[event viewer.png]]

Navigate to **Applications and Services Logs** -> **Microsoft** -> **Windows**.
Scroll down until you find the **Sysmon** folder
![[event viewer 1.png]]
![[event view 2.png]]


### Installing Wazuh

For the Wazuh and TheHive installation, we will be doing it on the **DigitalOcean** (https://www.digitalocean.com/) cloud provider.

DigitalOcean gives you a free $200 credit for 60 days whenever you use this link to sign up: [https://m.do.co/c/e2ce5a05f701](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbUE1X1ZjT0hjYnRsSmFYckpvNTV6S3pNdVpwUXxBQ3Jtc0tsci0xNC1Fb3M0ckhrSENyQTBoMkFPYTBiSHRGRGRlMDF6SHBZMktEYUp6SUd0NkhXRmpVRWNvQmZVRmlYTUg2czNTRkxsSnZjLURPWjl2aVFVYTRnSEV4UGVrUTZ1ZXI3T0hRYzU4ZmlJdU9vUWR2cw&q=https%3A%2F%2Fm.do.co%2Fc%2Fe2ce5a05f701&v=YxpUx0czgx4) , but a valid credit card is necessary. This is fair, 60 days is a lot of time to do any documentations regarding this project.

**IMPORTANT:** To avoid being billed after 60 days has ran out, you have to **DESTROY** the VMs created inside DigitalOcean after you have documented your project and/or are finished with the lab. 

To create a virtual machine, click on "Create" and select "Droplets". Virtual machines are called **Droplets** in DigitalOcean.
![[create droplet.png]]

For the **Region**, select the region closest to you. For the **image**, we will be using **Ubuntu 22.04 (LTS) x64**
![[droplet region image.png]]

For **CPU** options, select the **Basic plan**, then **Premium Intel**, then **8GB/ 2 Intel CPUs** option
![[droplet cpu.png]]

For the **authentication method**, choose a password and create a strong one.
![[droplet authentication.png]]

Name the hostname as "Wazuh", then click on **Create Droplet**
![[droplet name.png]]

### Creating a firewall
Inside DigitalOcean, we will be creating a firewall to protect our virtual machines against public scanners and unauthorized SSH, and to make sure that the VMs that we create in the cloud provider are only accessible to us via our own public IP address.

Before moving to the next step in creating a firewall, we need to identify our public IPv4 address first. you can go to https://www.whatismyip.com/ to find your public IP address.
![[whatismyip.png]]


In the DigitalOcean dashboard, click on the **Networking** link, then on the **Firewalls** tab, and click on **Create Firewall**
![[create firewall.png]]


For the inbound rules, create a rule where in it allows **All TCP** types on the TCP protocol, and **All ports** are allowed. Do the same for UDP. Then scroll down and click **Create Firewall**
![[wazuh inbound rules.png]]

We can now start adding VMs to the firewall we created.
![[droplet firewall created.png]]

In the dashboard, click on **Droplets**, and right then, we can see our Wazuh VM, along with its public IP address.
![[wazuh public ip.png]]
Click on Wazuh and click on the **Networking** tab.
![[wazuh networking.png]]Scroll down to the **Firewalls** section and click **Edit**
![[droplet firewall edit.png]]Select the firewall that was created earlier
![[droplet firewall 2.png]]On the **Droplets** tab, click **Add Droplets**
![[firewall add droplets.png]]
Search for your Wazuh VM then click **Add Droplet**
![[wazuh add firewall.png]]The Wazuh VM should now be protected by the firewall
![[wazuh firewall added.png]]

### Accessing the Wazuh VM

![[wazuh access droplet.png]]