**Goal:** The goal of this project is to create an automated SOC analyst lab complete with a SIEM tool, a case management tool, and a security automation platform for SOAR (Security Orchestration, Automation and Response) capabilities. This lab will also utilize **Mimikatz** as the sample credential harvesting attack.

# Tools to be used
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

#### Downloading Sysmon configuration file

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
![[SOC Automated Project/attachments/event viewer 1.png]]
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

In the **Droplets** dashboard, select your Wazuh VM, click on the **Access**, then select **Launch Droplet Console** to access the machine's terminal. You will be logged as **root**.
![[wazuh access droplet.png]]

For some reason, I couldn't access the terminal through the **Droplet console** throughout my entire lab activity. This may be caused by busy servers in Digital Ocean, or some other technical issue that I don't know about.

#### WSL (Windows Subsystem for Linux)

To remedy this, I installed **WSL (Windows Subsytem for Linux)** https://learn.microsoft.com/en-us/windows/wsl/install, since I am mainly using Windows 10 as my OS. 

To install WSL, open Powershell in **admin mode**, and enter:
```powershell
wsl --install
```

This will install the Ubuntu distro of Linux. This is enough to run commands and establish **ssh** connections with Digital Ocean VMs.

Once installed, you can now open an Ubuntu terminal within Windows.

#### SSH to Wazuh VM

To establish a connection with your Wazuh VM in Digital Ocean, open up a WSL terminal and make an **ssh** connection with the command:

`ssh root@MACHINE_IP`

The machine IP is your Wazuh VM's public IP address. Type your password as required, and you will be logged in as **root**

Update everything first by typing `apt-get update && apt-get upgrade -y`
![[wazuh terminal update.png]]

Just hit **enter** whenever you see these screens.
![[wazuh update.png]]
![[wazuh terminal update 2.png]]

After everything has updated, we can now run the Wazuh installer.

In the terminal, type: `curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a` and hit **enter**.

![[wazuh installer.png]]

Wazuh should complete its installation in a few moments, giving us this screen:
![[wazuh install complete.png]]
Take note of the credentials presented to you, since we will be using it when we log on to the browser interface, which be accessed by typing `https://[wazuh_public_ip]` in the browser

Click on **Advanced** when you see this screen
![[wazuh connection not private.png]]
then click on the bottom link to access the login page.
![[wazuh connection not private 2.png]]

We can now log in with the credentials we saved earlier.
![[wazuh login page.png]]


### Installing TheHive VM

The steps for installing TheHive VM is very similar to the steps we took in creating our Wazuh VM. 

![[thehive create droplet.png]]
![[droplet region image.png]]![[droplet cpu.png]]
![[droplet authentication.png]]

The only difference is the Hostname, which we will change to **thehive**.
![[droplet thehive name.png]]

After creating TheHive droplet, we should also add it to the firewall as we did with our Wazuh VM.
![[thehive add firewall.png]]
![[thehive add firewall 2.png]]

Now both our Wazuh and TheHive VM are protected by our firewall.
![[thehive wazuh firewall.png]]

### Accessing TheHive VM

In the **Droplets** dashboard, select your TheHive VM, click on the **Access,** then select **Launch Droplet Console** to access the machine's terminal. You will be logged as **root**.

![[thehive droplet console.png]]

Or, you can open up a WSL terminal and **ssh** to your TheHiveVM by:
`ssh root@MACHINE_IP` and provide your password.

**Note:** Don't forget to run `apt-get update && apt-get upgrade` first

#### Installing TheHive dependencies & applications

There are a lot of dependencies that we need to download and install in our TheHive VM. We just need to run the following commands in the terminal and we are good to go.

###### Dependencies
- `apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release`

###### Install Java
- `wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg`
- `echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list`
- `sudo apt update`
- `sudo apt install java-common java-11-amazon-corretto-jdk`
- `echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment `
- `export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"`

###### Install Cassandra
- `wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg`
- `echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list`
- `sudo apt update`
- `sudo apt install cassandra`

###### Install ElasticSearch
- `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg`
- `sudo apt-get install apt-transport-https`
- `echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list`
- `sudo apt update`
- `sudo apt install elasticsearch`
###### ***OPTIONAL ELASTICSEARCH***
- Create a **jvm.options** file under */etc/elasticsearch/jvm.options.d* and put the following configurations in that file:
```
-Dlog4j2.formatMsgNoLookups=true
-Xms2g
-Xmx2g
```

###### Install TheHive
- `wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg`
- `echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list`
- `sudo apt-get update`
- `sudo apt-get install -y thehive`

The default credentials for TheHive is:
- **username:** admin@thehive.local
- **password:** secret

# Part 3: Configuring Wazuh and TheHive

## Configuring TheHive

### Editing the Cassandra yaml file
**Cassandra**(https://cassandra.apache.org/_/index.html) is an open source database from **Apache** used by **TheHive**

We first need to configure the **cassandra.yaml** file for it to work in **TheHive** by customizing our listen ports and cluster name

**SSH** into **TheHive** terminal and type `nano /etc/cassandra/cassandra.yaml`
![[cassandra thehive ssh.png]]

Scroll down to the `cluster_name` and name it according to your preference

![[cassandra cluster name.png]]

Next, we will be changing the settings for the `listen_address` and `rpc_address`

To quickly search for strings inside Nano, press **ctrl+w** to bring up the search bar, type "**listen_address**" and press enter.

![[nano listen address.png]]

Scroll down to the `listen_address` setting and change the value from `localhost` to **TheHive's** public IP.

![[cassandra listen address.png]]

Same goes with searching for `rpc_address`.
![[nano rpc address.png]]

Scroll down to the `rpc_address` setting and change the value from `localhost` to **TheHive's** public IP.

![[cassandra rpc address.png]]

The last configuration we need to change is the **seed address**. Search for `seed_provider` and scroll down to the setting. Change the value from **127.0.0.1** to **TheHive's** public IP.
![[nano seed provider.png]]
![[cassandra seed provider.png]]
After that, press **ctrl+x** to prompt exit, type **Y** to save and press enter.

We will need to stop and restart **Cassandra's** services for the changes to take effect.

To stop **Cassandra**, type `systemctl stop cassandra.service` in the terminal and press enter.
![[stop cassandra.png]]

We also need to remove old files prior to configuring the yaml file. In the terminal, type `rm -rf /var/lib/cassandra/*`
**Note:** The ( * ) input will delete all files inside the directory.
![[remove old cassandra.png]]

To restart the service, type `systemctl start cassandra.service` in the terminal.
![[start cassandra.png]]

We should also maintain a good habit of double checking the status of the service we intend to run. To check **Cassandra's** status, type `systemctl status cassandra.service` in the terminal. It should say whether the service is active and running or has stopped or failed.
![[cassandra status.png]]
To exit from the status, press **q.**

### Setting up Elasticsearch

**Elasticsearch** is a search engine and data query management that is used by **TheHive.**

The configuration file for **Elasticsearch** is located at `/etc/elasticsearch/elasticsearch.yml`. To configure this, we need to open the **yml** file in **Nano.**

The first setting to change is the **cluster name**. Scroll down to the `cluster.name` value, remove the comment # and change the value to "**thehive**."

Scroll down to the `node.name` setting, remove the comment # and leave the value `node-1` as is.

Leave the values for `path.data` and `path.logs` as is.
![[elasticsearch yml.png]]

Scroll down to the `network.host` setting. Remove the comment, and change the value from the default to **TheHive's** public IP.

Remove the comment for `http.port`.

Remove the comment for `cluster.initial_master_nodes` and leave only `node-1` as the value.

![[elasticsearch config 2.png]]

Save the file and exit.

Now, we need to start **Elasticsearch** as a service. Type `systemctl start elasticsearch` in the terminal. 
![[start elasticsearch.png]]

We will also need to enable **Elasticsearch**. Type `systemctl enable elasticsearch`
![[enable elasticsearch.png]]

To double check the status, type `systemctl status elasticsearch`
![[elasticsearch status.png]]

We should also check **Cassandra's** status again after running **Elasticsearch** to make sure it is still running.
![[cassandra elasticsearch status.png]]

### Configuring TheHive config file

##### Changing file path ownership
Before configuring **TheHive's** config file, we need to check whether **TheHive's** user and its group has access to a particular file path. The file path is `/opt/thp`

To check permissions, type `ls -la /opt/thp`
![[thp access.png]]
Currently, the **root** user and group has access to the file path and needs to be changed.

To change ownership, type `chown -R thehive:thehive /opt/thp`.
![[chown thehive.png]]

Double check with `ls -la`.

![[thehive ownership.png]]

##### Setting up the config file.

**TheHive's** config file is located at `/etc/thehive/application.conf`

Open up the config file with **Nano**.

Scroll down the the first `hostname` setting and change the value from the default to **TheHive's** public IP.

Change the `cluster-name` value to the value that you put the **Cassandra** config file.

Within the `index.search` function, change the `hostname` from default to **TheHive's** public IP.

![[thehive config.png]]

Scrolling a bit down to the `Attachment storage configuration`, it states that the path should belong to the user and group running **thehive** service. This is the reason why we had to change ownership for the file path earlier.

![[thehive default ownership.png]]

Scroll down to the `Service configuration` section. Change the value of `application.baseUrl` from **localhost** to **TheHive's** public IP.
![[thehive service config.png]]
Save changes and exit the config file.

We now need to start and enable **TheHive** as a service with `systemctl start thehive` and `systemctl enable thehive`.
![[start enable thehive.png]]

We should also check **TheHive's** status with `systemctl status thehive`.
![[thehive status.png]]

**Important**: For **TheHive** to run successfully, it also needs **Cassandra** and **Elasticsearch** to be active and running as well. Double checking the status of all three is necessary.

![[status all.png]]

You can now access the web interface for **TheHive** by typing `http://thehive_publicip:9000` on the browser.

![[thehive log in page.png]]

You can also sign in with the default credentials of: `admin@thehive.local` and password: `secret`.

#### Troubleshooting TheHive login error

If you ever encounter an error when trying to log in to **TheHive**, it may be possible that the **Elasticsearch** service is down. You can check the status by `systemctl status elasticsearch`. 

If the error exists, **MyDIFR** recommends creating a custom **jvm** option file by typing `nano /etc/elasticsearch/jvm.options.d/jvm.optons` and put the following settings in the file:
```
-Dlog4j2.formatMsgNoLookups=true
-Xms2g
-Xmx2g
```
The settings above tells **TheHive** to allocate 2gb of memory for Java instead of the default (8gb).

## Configuring Wazuh

#### Logging in to the dashboard

We first need to access the **Wazuh** web interface by typing the **Wazuh** VM's public IP in the browser and logging in with the credentials that **Wazuh** has provided us during the installation section back in **Part 2**.

If you do not have the credentials provided, you can access the **Wazuh** manager terminal by **ssh** and type `ls`.
![[wazuh terminal.png]]
You should see the **.tar** file for wazuh install files, and we need to extract its contents by typing `tar -xvf wazuh-install-files.tar`.
![[wazuh install extract.png]]

After extracting, we can **cd** to the install files directory and see its contents. The file that we want is `wazuh-passwords.txt`
![[cd wazuh install.png]]

We can **cat** the .txt file to view its contents. We can now see the credentials we need to log in to the dashboard.
![[wazuh passwords.png]]

We should also take note of the credentials for **wazuh API user**, since we will be using it later on to perform responsive capabilities.
![[wazuh api user.png]]

#### Deploying a new agent

In the **Wazuh** dashboard, we can see that no agent has been added yet. To add one, simply click **Add agent**.

![[wazuh dashboard.png]]

In the **Deploy new agent** page, select **Windows** as the package. For the **Server address**, type in **Wazuh's** public IP. You can name the agent any way you want, but remember that it cannot be changed once the agent has been enrolled. Leave the groups tab in as default.

![[wazuh deploy agent.png]]

Copy the commands in the 4th step, and paste it in your Windows 10 machine's powershell terminal. This will install the Wazuh agent to your Windows 10 machine. Note that you should run powershell with admin privileges.

![[wazuh agent command.png]]

![[powershell wazuh agent.png]]

After the agent has finished installing, we can start the agent by typing `net start wazuhsvc`.

![[wazuh start service.png]]

You can also check the status of the wazuh agent by typing **"services"** in the start menu and opening the app. Then scroll down until you find **Wazuh** in the list.

![[services wazuh.png]]

Back in the **Wazuh** dashboard, wait for a few moments and refresh the page to see the enrolled agent.

![[wazuh dashboard agent.png]]
# Part 4: Generate Telemetry & Ingest into Wazuh

### Windows 10 Telemetry
We will be configuring the .conf file for **Wazuh** to generate telemetry and use **Mimikatz** as the sample attack .

In your Windows 10 VM, go to **C:\Program Files (x86)\ossec-agent** and look for the **ossec.conf** file. We will be configuring this config file to generate telemetry. 

Before opening the file, make a backup by copy-pasting the .conf file and renaming it **ossec-backup**.

You can open the .conf file in **Notepad** with **Administrative Privileges**.

We will be adding syntaxes right below this line group:

![[log analysis.png]]

You can also copy the syntax above and build rules from it. Paste the template just below the former syntax.

First, we will build a rule that will ingest **Sysmon** logs into **Wazuh**. We will be needing **Sysmon's** channel name for the `<location>` tag. **Sysmon's** channel name can be located via **Event Viewer**:

Open up **Event Viewer**, expand **Applications and Services Logs** -> **Microsoft** -> **Windows**, then scroll down to the **Sysmon** folder and click it. Right click on **Operational** and select **Properties**. 
![[event viewer 1 1.png]]
![[sysmon properties.png]]![[sysmon full name.png]]

Copy the **Full Name** and replace the `Application` value in the syntax.

![[sysmon channel name.png]]

For the sake of ingestion, we can remove the **Application**, **Security**, and **System** syntax and just retain the **Sysmon** syntax. This means that application, security, and system logs will not be forwarded into **Wazuh**. This is fine because we only want to observe **Sysmon** logs.

![[remove app security syntax.png]]
![[remove system syntax.png]]

Save the file and exit.

Now we need to restart **Wazuh** by going to Services -> **Wazuh** and clicking Restart.

![[restart wazuh.png]]

**Note:** Every time you change settings in the config file, you **MUST** restart the **Wazuh** service.

We can now go back to the **Wazuh** dashboard and search for "sysmon" events. It may take some time for events to show up.

![[wazuh sysmon search.png]]

### Downloading Mimikatz

**Mimikatz** is a credential harvesting application used by pentesters and hackers to extract credentials from a machine.

Before downloading **Mimikatz** we need to exclude our *Downloads* folder from Windows Security's scanning scope. 

Go to **Windows Security,** click on **Virus & threat protection** -> **Manage settings**, scroll down to the **Exclusions** section, and click **Add or remove exclusions**.

![[windows security.png]]
![[virus threat protection.png]]![[manage settings.png]]
![[exclusions.png]]

Click on **Add an exclusion** and choose "Folder", then browse to your *Downloads* folder and select it.
![[exclude download folder.png]]
![[select download folder.png]]

Your *Downloads* folder should appear on the list.
![[download folder list.png]]

Using **Google Chrome** is recommended when downloading **Mimikatz** because you can easily disable the browser protection from the security settings.

![[chrome settings.png]]
![[chrome no protection.png]]

**Mimikatz** download link: [https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919](https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919)

Right click on the **.zip** file and **Save As**, and save it to the *Downloads* folder.

![[download mimikatz.png]]
Go to your *Downloads* folder, right click on the zip file and select "Extract all"

![[extract mimikatz.png]]

Double click on the newly extracted folder and go inside the **x64** folder.

Open up **Powershell** with **admin** privileges and **cd** into the **x64** folder.

![[powershell mimikatz folder.png]]
Run `.\mimikatz.exe`.
![[mimikatz exe.png]]

Going back to the **Wazuh** dashboard, if you search for "mimikatz" events, you may not get anything yet. 

![[wazuh mimikatz alert 1.png]]

This is due to the fact that **Sysmon** or **Wazuh** rules may not be triggering alerts, because, by default, **Wazuh** will only log events whenever a rule or alert is triggered.

To fix this, we will need to configure the **ossec.conf** file in our **Wazuh manager** so that it will log everything, and/or create a rule or rules specifying a particular event so that **Wazuh** can log that event whenever that rule is triggered.

### Configuring the Wazuh manager config file

Go to the **Wazuh** manager terminal by accessing it via the Digital Ocean Droplet console or establishing an **ssh** connection.

The **ossec.conf** file is located at `/var/ossec/etc/ossec.conf`.

First, we'll create a backup copy of the config file by typing `cp /var/ossec/etc/ossec.conf ~/ossec-backup.conf`
![[wazuh linux ossec backup.png]]

Now we'll edit the .conf file in **Nano**: `nano /var/ossec/etc/ossec.conf`

Scroll down to the `<logall>` and `<logall_json>` options and change the value from 'no' to 'yes'

![[wazuh manager nano.png]]

Save and exit the file.

Now we need to restart the **Wazuh** service by `systemctl restart wazuh-manager.service`.

By then, **Wazuh** will now log all events and put it in a file called "**archives**", which is located at `/var/ossec/logs/archives`.

![[wazuh archives 1.png]]

In order for **Wazuh** to start ingesting the logs, we need to configure our **filebeat** file. **Filebeat** is a log forwarder used by **Elasticsearch**. 

We can edit the **filebeat** config file in **nano** by `nano /etc/filebeat/filebeat.yml`.

Scroll down to the `filebeat.modules` option, and change the `enabled` value under `archives` from false to `true`.

![[filebeat true.png]]

Save the file and exit.

Restart the **filebeat** service by `systemctl restart filebeat`.

### Creating a new Wazuh index

In our **Wazuh** dashboard, click on the **3-line icon**, click on **Stack Management**, and select **Index Patterns**.
![[wazuh stack management.png]]
![[index patterns.png]]

Inside **Index patterns**, we will originally see three patterns.
![[index pattern list.png]]

We will need to create a new index pattern for our **archives**. Click on the **Create index pattern** and name the new pattern as `wazuh-archives-**`

![[create new index.png]]

![[wazuh archives name.png]]
Click on **Next**, then select `timestamp` for the **Time field**, then click **Create index pattern**.
![[timestamp time field.png]]

To make `wazuh-archives-**` our default index pattern, click on the **3-line icon** again and select **Discover**.
![[discover.png]]

Then click the dropdown button and select `wazuh-archives-**`.

![[wazuh archive default.png]]
It will take time before events start to flow in, but eventually they will come.

You can also re-run **Mimikatz** in the Windows 10 client to see if any alerts appear in **Wazuh**.

In the **Mimikatz** terminal, exit from the first run by typing `exit` and running it again.
![[mimikatz rerun.png]]

Back in the **Wazuh** dashboard, search for "mimikatz" events again, and a couple of alerts should appear. We should focus more on the alert having an **Event ID** of "1", because it is the event ID for **process creation**. 
![[mimikatz alert.png]]

Expanding the alert for more details, we can scroll down and see a field called `originalFileName`. We can use this field to create our alert for **Mimikatz**. This field is more reliable to use than the `image` field, because attackers can simply rename the file and it will reflect on that field, and they can bypass the alert, whereas the `originalFileName` field keeps the original name of that file, regardless of any renaming process.

![[orignalfilename mimikatz.png]]


### Creating Rules in Wazuh

In the **Wazuh** dashboard, click on the drop-down button, click on **Management**, and select **Rules**.
![[wazuh create rules.png]]

Click on **Manage Rules Files**.
![[manage rules files.png]]

Search for "sysmon" and look for **0800-sysmon_id_1.xml**. Preview the file by clicking on the **eye** icon

![[sysmon event id 1.png]]

We can copy the first rule and use it as our template.
![[sysmon copy rule.png]]

Go back from the sysmon xml rule and click **Custom rules**.

![[custom rules.png]]

Inside, we can see the `local_rules.xml` file. We can edit this by clicking on the **pencil** icon.
![[local rules.png]]

We can paste the rule we copied earlier below the already existing rule inside **local_rules**.

**Note:** We must ensure and follow proper identations and naming of values in order for the rules to be applied successfully.

- By default, rule IDs start from **100000**, so we'll change the `rule id` to **100002**, since there is already another rule above us.
- The `level`'s max range is **15**, so we'll just set it to max.
- For the `field name`, we'll set it to `win.eventdata.originalFileName`, then change `\\(c|w)script\.exe` to `mimikatz\.exe` inside the `type` field.
- Remove the `<options>` field, then change the `<description>` to `Mimikatz Usage Detected`.
- Change the **mirtre id** to `T1003`,  (**credential dumping**), which is what **Mimikatz** is known to do.

![[create mimikatz rule.png]]

Save the rule and restart the manager.

Before we run **Mimikatz** again, we can try renaming the .exe file into something different.
![[rename mimikatz.png]]

Now we'll run **Mimikatz** in powershell once more.

![[verylegitprogram.png]]

Refresh the **Wazuh** dashboard, then go to **Security Events** to check if any alert has been triggered.

![[mimikatz alert triggered.png]]![[mimikatz original name.png]]

Even if we renamed **Mimikatz** into something else, the alert still triggered because we used the `originalFileName` value.

# Part 5: Connect Suffle (SOAR), Send Alert to TheHive, Send to SOC Analyst via Email

### Configuring Shuffle
Head over to the **Shuffle** site (https://shuffler.io) and create an account

Click on **Workflows**
![[shuffle workflow.png]]

Click on **New Workflow** to create a new workflow
![[shuffle new workflow.png]]

Name the workflow as "**SOC Automation Project**" and type any description that you like. For the **Usecases**, you can select any in the list. Click **Done** when you're ready.
![[shuffle name workflow.png]]

This will be your new dashboard
![[shuffle dashboard.png]]

Navigate to **Triggers** on the bottom left corner, then drag and drop the **Webhook** app into the dashboard.
![[shuffle add webhook.png]]

Name the webhook as "**Wazuh-Alerts**", then copy the **Webhook URI**. We will need this URI to put into the **ossec.conf** file in our **Wazuh** manager.
![[shuffle name webhook.png]]

Click on the **Change Me** icon, select the **Repeat back to me** option in the **Find Actions** tab, and on the **Call** tab, click on the plus icon and select **Execution Argument**.
![[changeme settings.png]]

Save the changes.
![[changeme save.png]]

#### Integrating Shuffle to Wazuh

We'll head over to the **Wazuh** manager terminal to connect **Shuffle** with **Wazuh**. We will be using an **integration** tag inside the **ossec.conf** file.

Open the **ossec.conf** file with **nano**. Scroll down just past below the `<global>` tag and paste the following tag:

```
<integration>
  <name>shuffle</name>
  <hook_url><YOUR_SHUFFLE_URL</hook_url>
  <rule_id>1000002</rule_id>
  <alert_format>json</alert_format>
</integration>
```

![[shuffle integration.png]]

Replace `YOUR_SHUFFLE_URL` with the url that we copied from **Shuffle** earlier. Add a space (  ) between the url and the `</hook_url>` closing tag to avoid the tag becoming a part of the url. This is indicated by having the url highlighted, but not the `</hook_url>` tag.

The `<rule_id>` is the rule ID for **Mimikatz** that we configured earlier in the **Wazuh** custom rules.

Save and exit the editor.

Restart the wazuh manager service by `systemctl restart wazuh-manager.service` and check the status by `systemctl status wazuh-manager.service`

#### Testing the integration

In your Windows 10 VM, re-run **Mimikatz** to generate telemetry.

In the **Shuffle** dashboard, click on the **Wazuh-Alerts** webhook and click **Start**. Then click on the running man icon at the bottom to show executions.

![[shuffle test telemetry.png]]

This should show results regarding the telemetry. Click on the result and expand the **Execution Argument** tab.
![[shuffle telemetry result.png]]

This shows us the information that is generated from **Wazuh**.
![[shuffle info from wazuh.png]]

Before moving on with the **IOC Enrichment** setup, we'll review our current objectives for the workflow:
1. **Mimikatz** alert sent to **Shuffle**
2. **Shuffle** receives **Mimikatz** alert and extracts **SHA256** hash from file
3. Check reputation score with **VirustTotal**
4. Send details to **TheHive** to create alert
5. Send email to **SOC Analyst** to begin investigation
### IOC Enrichment

![[hash value.png]]


The file's return value for the **hashes** is appended by their hash type (e.g., **SHA1, MD5**). We will need to parse out the hash value itself to be sent to VirusTotal.


Click on the **Change Me** icon, and search for "**Regex capture group**" in the **Find Actions** tab, and select it.
![[regex capture group.png]]

In the **Input data**, select **Execution Argument** and find the **hashes** parameter. You can check the values of a parameter by hovering over it and looking at the details that appear beside it.
![[input data.png]]
![[execution argument hashes.png]]

For the **Regex** field, we can utilize **ChatGPT** to help us create the formula for the hashes. We'll type a prompt to create a regex to parse our file's sha256 value. Copy the hash details of the file earlier and paste it in the prompt as seen below.

![[regex field.png]]
![[chatgpt hash prompt.png]]

We can now copy the regex that **ChatGPT** has created after it has finished generating. 
![[Pasted image 20240119170417.png]]
P