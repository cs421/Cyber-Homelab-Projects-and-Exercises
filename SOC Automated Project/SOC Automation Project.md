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
Open the **Event Viewer** by typing 'event' in the start menu and cli
![[Pasted image 20240112164818