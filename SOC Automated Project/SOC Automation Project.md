**Goal:** The goal of this project is to create an automated SOC analyst lab complete with a SIEM tool, a case management tool, and a security automation platform for SOAR (Security Orchestration, Automation and Response) capabilities. This lab will also utilize **Mimikatz** as the sample credential harvesting attack.

# Tools to be used
1. **Wazuh** (https://wazuh.com) - XDR and SIEM tool, for receiving events and sending alerts to SOAR
2. **Shuffle** (https://shuffler.io) - for SOAR and IOC (indicators of compromise) enrichment
3. **TheHive** (https://thehive-project.org) - Security Incident Response Platform, for ticket and alert assignments to SOC analysts
4. **VirusTotal** (https://www.virustotal.com) - for IOC enrichment and threat intel
5. **Mimikatz** - (https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919) for sample credential harvesting attack

There will be **two (2)** virtual machines to be used in this project, namely:
1. A Windows 10 Client which will serve as the Wazuh agent, to be configured via VirtualBox.
2. An Ubuntu machine which will serve as another Wazuh agent and for testing active response capabilities later on, to be configured on the cloud.

**Wazuh, TheHive, and Shuffle** will be configured on the cloud.

I will be following **MyDFIR's** videos in creating this lab, as found in: https://www.youtube.com/watch?v=Lb_ukgtYK_U

---

# Part 1: Diagram

![SOC Automated Homelab Diagram](https://github.com/cs421/Create_Homelab_Project/assets/152476259/712059be-5d9b-47b8-9490-74bc760df2f1)

**1.** The  **Wazuh** agent (Windows 10 client) will send events to the internet.

**2.** The **Wazuh** manager will then receive those events sent by the client and will create alerts.

**3.** **Wazuh** will send the alerts to **Shuffle**

**4.** **Shuffle** will send the alerts over to the Internet for IOC enrichment via OSINT (Open Source Intelligence), then back to **Shuffle**

**5.** **Shuffle** will then send the alerts over to **TheHive** for case management.

**6.** **Shuffle** will also send and email to the SOC analyst regarding the alerts

**7.** The SOC analyst will receive the email sent by **Shuffle**. The email should contain details of the alert, and will ask the question, "Do you want to contain this event or not?" to the SOC analyst.

**8.** The SOC analyst will send the appropriate response action to **Shuffle**, which, in turn, will send the response back to the **Wazuh**.

**9.** **Wazuh** will instruct the agent to perform the appropriate responsive action.


# Part 2: Installing Virtual Machines and Applications

### Installing VirtualBox and Windows 10 virtual machine

Installing VirtualBox and the Windows 10 VM is very linear, as shown in MyDFIR's tutorial: https://www.youtube.com/watch?v=YxpUx0czgx4 from **2:07 - 8:34**
### Installing Sysmon
By now, the Windows 10 VM should have been installed and prepped. Open up a browser and go to https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon and choose **Download Sysmon**

![download sysmon](https://github.com/cs421/Create_Homelab_Project/assets/152476259/3893b005-09bc-4eee-abe9-2f3647a60d49)

#### Downloading Sysmon configuration file

Go to https://github.com/olafhartong/sysmon-modular, scroll down and click **sysmonconfig.xml**

![sysmon config](https://github.com/cs421/Create_Homelab_Project/assets/152476259/88ee7dfd-cac4-412e-bbad-a14ef1a4db22)

click **Raw**, then right click and **Save as** the xml file

![sysmon config raw](https://github.com/cs421/Create_Homelab_Project/assets/152476259/de2b7769-40c8-4257-bfca-2a1945035887)
![sysmon xml save](https://github.com/cs421/Create_Homelab_Project/assets/152476259/71a64859-cf72-4ec6-bff5-daedb4e3cf8f)

Go to the Sysmon's download location, right click the zip file and select "Extract All"

![sysmon extract](https://github.com/cs421/Create_Homelab_Project/assets/152476259/a8eccb03-3437-4a95-bc6a-d3ff142246b0)


Move or drag and drop the sysmon config file into the sysmon folder.

![sysmonconfig move](https://github.com/cs421/Create_Homelab_Project/assets/152476259/4121ce1c-bc1b-44a6-b9a3-bcff77733195)


We will be needing **Powershell** in this next step. Go to start, and search for powershell, and select "Run as Administrator"

![powershell admin](https://github.com/cs421/Create_Homelab_Project/assets/152476259/06416e51-13f3-4c5e-9788-7fc78d169ea8)

`cd` to the extracted sysmon folder

![sysmon folder](https://github.com/cs421/Create_Homelab_Project/assets/152476259/79e49b86-8ac9-423c-8d18-875af89dc23d)

**Note:** For good practice, always put paths inside quotation marks (" ")

We will now install sysmon along with the downloaded configuration file.

Run `.\sysmon64.exe -i sysmonconfig.xml`

![sysmon64 install](https://github.com/cs421/Create_Homelab_Project/assets/152476259/04c32262-e04d-4c80-8bb3-4c446338bd51)


The license agreement window should appear. Click "Agree", and let the installer run its course

![sysmon64 license agreement](https://github.com/cs421/Create_Homelab_Project/assets/152476259/d24ae347-d353-42f0-bf1a-84b366c0f868)

![sysmon install progress](https://github.com/cs421/Create_Homelab_Project/assets/152476259/38bfa904-93a0-49d6-960e-69ac9ead15f9)

To double check if Sysmon has been successfully installed, we can do the following steps:

##### Services
Open the **Services** app by typing 'services' in the start menu

![services](https://github.com/cs421/Create_Homelab_Project/assets/152476259/3eb14450-1171-46e6-88d0-958e63234ec3)

Scroll down until you find **Sysmon64** in the list.

![services sysmon](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ece9d54b-eaa6-4e7f-8085-f3dca74c803e)


##### Event Viewer
Open the **Event Viewer** by typing 'event' in the start menu and clicking on the app

![event viewer](https://github.com/cs421/Create_Homelab_Project/assets/152476259/5bbe9b74-2e01-46c0-a1a3-eb4171b3821c)


Navigate to **Applications and Services Logs** -> **Microsoft** -> **Windows**.
Scroll down until you find the **Sysmon** folder

![event viewer 1](https://github.com/cs421/Create_Homelab_Project/assets/152476259/50c53bf8-a934-4975-97e8-10b86c37376f)

![event view 2](https://github.com/cs421/Create_Homelab_Project/assets/152476259/0330ab81-c093-418c-b055-5eaea0366a51)


### Installing Wazuh

For the Wazuh and TheHive installation, we will be doing it on the **DigitalOcean** (https://www.digitalocean.com/) cloud provider.

DigitalOcean gives you a free $200 credit for 60 days whenever you use this link to sign up: [https://m.do.co/c/e2ce5a05f701](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbUE1X1ZjT0hjYnRsSmFYckpvNTV6S3pNdVpwUXxBQ3Jtc0tsci0xNC1Fb3M0ckhrSENyQTBoMkFPYTBiSHRGRGRlMDF6SHBZMktEYUp6SUd0NkhXRmpVRWNvQmZVRmlYTUg2czNTRkxsSnZjLURPWjl2aVFVYTRnSEV4UGVrUTZ1ZXI3T0hRYzU4ZmlJdU9vUWR2cw&q=https%3A%2F%2Fm.do.co%2Fc%2Fe2ce5a05f701&v=YxpUx0czgx4) , but a valid credit card is necessary. This is fair, 60 days is a lot of time to do any documentations regarding this project.

**IMPORTANT:** To avoid being billed after 60 days has ran out, you have to **DESTROY** the VMs created inside DigitalOcean after you have documented your project and/or are finished with the lab. 

To create a virtual machine, click on "Create" and select "Droplets". Virtual machines are called **Droplets** in DigitalOcean.

![create droplet](https://github.com/cs421/Create_Homelab_Project/assets/152476259/1c02572f-cd45-4d96-87df-0ec04efa794d)


For the **Region**, select the region closest to you. For the **image**, we will be using **Ubuntu 22.04 (LTS) x64**

![droplet region image](https://github.com/cs421/Create_Homelab_Project/assets/152476259/b8e8391f-ea0a-45a9-ae24-2e63122f8bee)


For **CPU** options, select the **Basic plan**, then **Premium Intel**, then **8GB/ 2 Intel CPUs** option

![droplet cpu](https://github.com/cs421/Create_Homelab_Project/assets/152476259/62d16fd5-429e-4f55-ab8f-7d623f8d6c82)


For the **authentication method**, choose a password and create a strong one.

![droplet authentication](https://github.com/cs421/Create_Homelab_Project/assets/152476259/f8d353b9-608f-4c6d-bc91-8dc16c4cc2aa)


Name the hostname as "Wazuh", then click on **Create Droplet**

![droplet name](https://github.com/cs421/Create_Homelab_Project/assets/152476259/a02334bb-94ec-4a94-997a-fd281f60862a)


### Creating a firewall
Inside DigitalOcean, we will be creating a firewall to protect our virtual machines against public scanners and unauthorized SSH, and to make sure that the VMs that we create in the cloud provider are only accessible to us via our own public IP address.

Before moving to the next step in creating a firewall, we need to identify our public IPv4 address first. you can go to https://www.whatismyip.com/ to find your public IP address.

![whatismyip](https://github.com/cs421/Create_Homelab_Project/assets/152476259/7b4e956d-3d3d-49fc-90ee-9737aee38f1b)


In the DigitalOcean dashboard, click on the **Networking** link, then on the **Firewalls** tab, and click on **Create Firewall**

![create firewall](https://github.com/cs421/Create_Homelab_Project/assets/152476259/b19ea106-1bda-4e8f-bfb9-cb572b3786a8)


For the inbound rules, create a rule where in it allows **All TCP** types on the TCP protocol, and **All ports** are allowed. Do the same for UDP. Then scroll down and click **Create Firewall**

![wazuh inbound rules](https://github.com/cs421/Create_Homelab_Project/assets/152476259/f69a6cc6-3e5d-4899-9565-5cfce71e80ab)


We can now start adding VMs to the firewall we created.

![droplet firewall created](https://github.com/cs421/Create_Homelab_Project/assets/152476259/a512f582-b438-42b0-b71c-56dd465a6a76)


In the dashboard, click on **Droplets**, and right then, we can see our Wazuh VM, along with its public IP address.

![wazuh public ip](https://github.com/cs421/Create_Homelab_Project/assets/152476259/149b03b4-92c0-4098-978e-309f3e2f30ba)

Click on Wazuh and click on the **Networking** tab.

![wazuh networking](https://github.com/cs421/Create_Homelab_Project/assets/152476259/52384a10-b5b1-41c4-ab1c-850160d82fb6)

Scroll down to the **Firewalls** section and click **Edit**

![droplet firewall edit](https://github.com/cs421/Create_Homelab_Project/assets/152476259/2e993244-258b-4714-9cfd-1abdc798279b)

Select the firewall that was created earlier

![droplet firewall 2](https://github.com/cs421/Create_Homelab_Project/assets/152476259/26ec5b85-2a29-46ae-b29f-211e6cb58903)

On the **Droplets** tab, click **Add Droplets**

![firewall add droplets](https://github.com/cs421/Create_Homelab_Project/assets/152476259/26a1cd44-e601-4aed-b03c-913619ca089b)

Search for your Wazuh VM then click **Add Droplet**

![wazuh add firewall](https://github.com/cs421/Create_Homelab_Project/assets/152476259/6d968696-27b7-45cf-b972-f0515ea45c15)

The Wazuh VM should now be protected by the firewall

![wazuh firewall added](https://github.com/cs421/Create_Homelab_Project/assets/152476259/0c7158cc-03dc-4892-9231-cfcc3113290d)

### Accessing the Wazuh VM

In the **Droplets** dashboard, select your Wazuh VM, click on the **Access**, then select **Launch Droplet Console** to access the machine's terminal. You will be logged as **root**.

![wazuh access droplet](https://github.com/cs421/Create_Homelab_Project/assets/152476259/88fb2b01-002a-4c2f-b84f-e5c6ddef0862)


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

![wazuh terminal update](https://github.com/cs421/Create_Homelab_Project/assets/152476259/b7e1259e-1167-409b-8e08-c63992438c79)

Just hit **enter** whenever you see these screens.

![wazuh update](https://github.com/cs421/Create_Homelab_Project/assets/152476259/5a203a9d-9e3c-41ce-b8f0-06d6f134aaeb)

![wazuh terminal update 2](https://github.com/cs421/Create_Homelab_Project/assets/152476259/892ead4b-6614-44c8-b0de-f13fa6b2952e)


After everything has updated, we can now run the Wazuh installer.

In the terminal, type: `curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a` and hit **enter**.

![wazuh installer](https://github.com/cs421/Create_Homelab_Project/assets/152476259/eadcfae4-87b2-4168-876f-af1896c7bfb0)

Wazuh should complete its installation in a few moments, giving us this screen:

![wazuh install complete](https://github.com/cs421/Create_Homelab_Project/assets/152476259/8ceedb81-3b9d-4845-bff2-98f5e197af9d)

Take note of the credentials presented to you, since we will be using it when we log on to the browser interface, which be accessed by typing `https://[wazuh_public_ip]` in the browser

Click on **Advanced** when you see this screen

![wazuh connection not private](https://github.com/cs421/Create_Homelab_Project/assets/152476259/4172af0e-79a1-46b1-86a4-faa3428a529e)

then click on the bottom link to access the login page.

![wazuh connection not private 2](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ea706eea-7e90-4e3a-9935-efb0085782a6)

We can now log in with the credentials we saved earlier.
![wazuh login page](https://github.com/cs421/Create_Homelab_Project/assets/152476259/1922e017-e7cc-47c2-811a-b0f514467ed7)


### Installing TheHive VM

The steps for installing TheHive VM is very similar to the steps we took in creating our Wazuh VM. 

![droplet region image](https://github.com/cs421/Create_Homelab_Project/assets/152476259/352a5ca3-69bb-4400-8d65-7d57862938cd)
![droplet cpu](https://github.com/cs421/Create_Homelab_Project/assets/152476259/6e6d6563-4e96-4331-811c-1adf571b5816)
![droplet authentication](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ce890d54-1b43-4862-babb-9231c7e772c8)

The only difference is the Hostname, which we will change to **thehive**.
![droplet thehive name](https://github.com/cs421/Create_Homelab_Project/assets/152476259/c36c8c45-7d69-4da4-b813-9a09b58c7744)


After creating TheHive droplet, we should also add it to the firewall as we did with our Wazuh VM.
![thehive add firewall](https://github.com/cs421/Create_Homelab_Project/assets/152476259/e9c6a995-3007-4f86-af6d-aabf27aec46f)

![thehive add firewall 2](https://github.com/cs421/Create_Homelab_Project/assets/152476259/cadd6107-045f-4e0c-a038-c75248d18692)

Now both our Wazuh and TheHive VM are protected by our firewall.

![thehive wazuh firewall](https://github.com/cs421/Create_Homelab_Project/assets/152476259/5aee2eb3-73cc-4961-8355-d60222b3a390)


### Accessing TheHive VM

In the **Droplets** dashboard, select your TheHive VM, click on the **Access,** then select **Launch Droplet Console** to access the machine's terminal. You will be logged as **root**.

![thehive droplet console](https://github.com/cs421/Create_Homelab_Project/assets/152476259/7626c286-fc62-487e-9eff-684fb14fe403)

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

![cassandra thehive ssh](https://github.com/cs421/Create_Homelab_Project/assets/152476259/86872cac-4f0e-4f25-b153-30d455fbf4fd)

Scroll down to the `cluster_name` and name it according to your preference

![cassandra cluster name](https://github.com/cs421/Create_Homelab_Project/assets/152476259/73e77281-22b1-4208-80f9-a7691090a6de)

Next, we will be changing the settings for the `listen_address` and `rpc_address`

To quickly search for strings inside Nano, press **ctrl+w** to bring up the search bar, type "**listen_address**" and press enter.

![nano listen address](https://github.com/cs421/Create_Homelab_Project/assets/152476259/4964310d-b8b5-46f7-8536-b5c11573bfb6)

Scroll down to the `listen_address` setting and change the value from `localhost` to **TheHive's** public IP.

![cassandra listen address](https://github.com/cs421/Create_Homelab_Project/assets/152476259/7843da7c-4019-421f-9403-9f0422b5ee07)

Same goes with searching for `rpc_address`.

![nano rpc address](https://github.com/cs421/Create_Homelab_Project/assets/152476259/61926d5f-f90c-4e00-8996-94bddf7bb621)

Scroll down to the `rpc_address` setting and change the value from `localhost` to **TheHive's** public IP.

![cassandra rpc address](https://github.com/cs421/Create_Homelab_Project/assets/152476259/2b105e0a-0d3c-4c9a-93be-915d09ea5286)

The last configuration we need to change is the **seed address**. Search for `seed_provider` and scroll down to the setting. Change the value from **127.0.0.1** to **TheHive's** public IP.

![nano seed provider](https://github.com/cs421/Create_Homelab_Project/assets/152476259/54e58991-f6cb-4dac-93b2-c781363b2616)

![cassandra seed provider](https://github.com/cs421/Create_Homelab_Project/assets/152476259/716f7f12-ac0c-422c-8e3a-9b076964055a)

After that, press **ctrl+x** to prompt exit, type **Y** to save and press enter.

We will need to stop and restart **Cassandra's** services for the changes to take effect.

To stop **Cassandra**, type `systemctl stop cassandra.service` in the terminal and press enter.

![stop cassandra](https://github.com/cs421/Create_Homelab_Project/assets/152476259/bae64686-d136-4aa1-a6e8-899e2eaac6d0)

We also need to remove old files prior to configuring the yaml file. In the terminal, type `rm -rf /var/lib/cassandra/*`
**Note:** The ( * ) input will delete all files inside the directory.

![remove old cassandra](https://github.com/cs421/Create_Homelab_Project/assets/152476259/8319b949-955c-435f-a767-e304e62a87ac)

To restart the service, type `systemctl start cassandra.service` in the terminal.

![start cassandra](https://github.com/cs421/Create_Homelab_Project/assets/152476259/1e43fc73-4808-4455-874b-da2cf3d81a0b)

We should also maintain a good habit of double checking the status of the service we intend to run. To check **Cassandra's** status, type `systemctl status cassandra.service` in the terminal. It should say whether the service is active and running or has stopped or failed.

![cassandra status](https://github.com/cs421/Create_Homelab_Project/assets/152476259/4846fb2a-017d-4f8b-a843-822a712af716)

To exit from the status, press **q.**

### Setting up Elasticsearch

**Elasticsearch** is a search engine and data query management that is used by **TheHive.**

The configuration file for **Elasticsearch** is located at `/etc/elasticsearch/elasticsearch.yml`. To configure this, we need to open the **yml** file in **Nano.**

The first setting to change is the **cluster name**. Scroll down to the `cluster.name` value, remove the comment # and change the value to "**thehive**."

Scroll down to the `node.name` setting, remove the comment # and leave the value `node-1` as is.

Leave the values for `path.data` and `path.logs` as is.

![elasticsearch yml](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ce04509c-d6da-4006-9fef-1fd8a6909f7d)

Scroll down to the `network.host` setting. Remove the comment, and change the value from the default to **TheHive's** public IP.

Remove the comment for `http.port`.

Remove the comment for `cluster.initial_master_nodes` and leave only `node-1` as the value.

![elasticsearch config 2](https://github.com/cs421/Create_Homelab_Project/assets/152476259/b84d4148-3123-4274-add5-54bcb246acdb)

Save the file and exit.


Now, we need to start **Elasticsearch** as a service. Type `systemctl start elasticsearch` in the terminal. 

![start elasticsearch](https://github.com/cs421/Create_Homelab_Project/assets/152476259/23162864-1239-4d87-b388-400477fad4e1)

We will also need to enable **Elasticsearch**. Type `systemctl enable elasticsearch`

![enable elasticsearch](https://github.com/cs421/Create_Homelab_Project/assets/152476259/bcf1c76c-df35-41f8-a38b-6812db677315)


To double check the status, type `systemctl status elasticsearch`

![elasticsearch status](https://github.com/cs421/Create_Homelab_Project/assets/152476259/90012914-5aab-4860-8c2e-292304353adb)

We should also check **Cassandra's** status again after running **Elasticsearch** to make sure it is still running.

![cassandra elasticsearch status](https://github.com/cs421/Create_Homelab_Project/assets/152476259/9949a75e-1f09-42c0-9d69-614c15467588)


### Configuring TheHive config file

##### Changing file path ownership
Before configuring **TheHive's** config file, we need to check whether **TheHive's** user and its group has access to a particular file path. The file path is `/opt/thp`

To check permissions, type `ls -la /opt/thp`

![thp access](https://github.com/cs421/Create_Homelab_Project/assets/152476259/64b3e18e-bee5-427a-9c52-b47c1e9df18c)

Currently, the **root** user and group has access to the file path and needs to be changed.

To change ownership, type `chown -R thehive:thehive /opt/thp`.

![chown thehive](https://github.com/cs421/Create_Homelab_Project/assets/152476259/695d60f7-dbaf-4c15-a3a8-3f45a4ff7493)

Double check with `ls -la`.

![thehive ownership](https://github.com/cs421/Create_Homelab_Project/assets/152476259/7d146cc6-66f5-437c-9ec0-a191a1118128)


##### Setting up the config file.

**TheHive's** config file is located at `/etc/thehive/application.conf`

Open up the config file with **Nano**.

Scroll down the the first `hostname` setting and change the value from the default to **TheHive's** public IP.

Change the `cluster-name` value to the value that you put the **Cassandra** config file.

Within the `index.search` function, change the `hostname` from default to **TheHive's** public IP.

![thehive config](https://github.com/cs421/Create_Homelab_Project/assets/152476259/3088a893-8b5b-45f8-9aff-4e6b4c41197e)

Scrolling a bit down to the `Attachment storage configuration`, it states that the path should belong to the user and group running **thehive** service. This is the reason why we had to change ownership for the file path earlier.

![thehive default ownership](https://github.com/cs421/Create_Homelab_Project/assets/152476259/0d22ae42-782d-4fc6-8f4e-b78e0ac4e16f)

Scroll down to the `Service configuration` section. Change the value of `application.baseUrl` from **localhost** to **TheHive's** public IP.

![thehive service config](https://github.com/cs421/Create_Homelab_Project/assets/152476259/913767e8-7650-425c-9c83-5295cf182a70)

Save changes and exit the config file.


We now need to start and enable **TheHive** as a service with `systemctl start thehive` and `systemctl enable thehive`.

![start enable thehive](https://github.com/cs421/Create_Homelab_Project/assets/152476259/fff44300-f5a3-4c5c-87fb-d7bb6b15f8b2)

We should also check **TheHive's** status with `systemctl status thehive`.

![thehive status](https://github.com/cs421/Create_Homelab_Project/assets/152476259/89fa38a0-7f53-4614-8abf-de1375afea95)


**Important**: For **TheHive** to run successfully, it also needs **Cassandra** and **Elasticsearch** to be active and running as well. Double checking the status of all three is necessary.

![status all](https://github.com/cs421/Create_Homelab_Project/assets/152476259/9273aacd-1f05-407b-bb9b-9a2dbeec6c20)

You can now access the web interface for **TheHive** by typing `http://thehive_publicip:9000` on the browser.

![thehive log in page](https://github.com/cs421/Create_Homelab_Project/assets/152476259/67f520b4-d1b8-4d41-91e2-45fa098a97a9)

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

![wazuh terminal](https://github.com/cs421/Create_Homelab_Project/assets/152476259/89278a04-c3e0-4d64-a0e6-9d5d7ce348b6)

You should see the **.tar** file for wazuh install files, and we need to extract its contents by typing `tar -xvf wazuh-install-files.tar`.

![wazuh install extract](https://github.com/cs421/Create_Homelab_Project/assets/152476259/0607d196-f92b-4ca4-8932-d309ef869505)

After extracting, we can **cd** to the install files directory and see its contents. The file that we want is `wazuh-passwords.txt`

![cd wazuh install](https://github.com/cs421/Create_Homelab_Project/assets/152476259/f67157a6-8922-4403-b9a9-3c11996544bd)

We can **cat** the .txt file to view its contents. We can now see the credentials we need to log in to the dashboard.

![wazuh passwords](https://github.com/cs421/Create_Homelab_Project/assets/152476259/39ab3455-10c9-4173-ba2f-86fbe41e6916)


We should also take note of the credentials for **wazuh API user**, since we will be using it later on to perform responsive capabilities.

![wazuh api user](https://github.com/cs421/Create_Homelab_Project/assets/152476259/f69dd609-b0cd-403a-9597-6995caf19dd4)


#### Deploying a new agent

In the **Wazuh** dashboard, we can see that no agent has been added yet. To add one, simply click **Add agent**.

![wazuh dashboard](https://github.com/cs421/Create_Homelab_Project/assets/152476259/a3c020bf-be62-42cc-85e9-0d94243fa71b)

In the **Deploy new agent** page, select **Windows** as the package. For the **Server address**, type in **Wazuh's** public IP. You can name the agent any way you want, but remember that it cannot be changed once the agent has been enrolled. Leave the groups tab in as default.

![wazuh deploy agent](https://github.com/cs421/Create_Homelab_Project/assets/152476259/e4ba8875-6fbb-4d48-ad1b-d5efdf087c02)

Copy the commands in the 4th step, and paste it in your Windows 10 machine's powershell terminal. This will install the Wazuh agent to your Windows 10 machine. Note that you should run powershell with admin privileges.

![wazuh agent command](https://github.com/cs421/Create_Homelab_Project/assets/152476259/6d593468-6636-49ad-a426-03a04c862edd)

![powershell wazuh agent](https://github.com/cs421/Create_Homelab_Project/assets/152476259/47174bf0-ee33-41dd-aae1-ee3118518c39)

After the agent has finished installing, we can start the agent by typing `net start wazuhsvc`.

![wazuh start service](https://github.com/cs421/Create_Homelab_Project/assets/152476259/66caa96f-7d59-4e22-acb6-205639e97b67)

You can also check the status of the wazuh agent by typing **"services"** in the start menu and opening the app. Then scroll down until you find **Wazuh** in the list.

![services wazuh](https://github.com/cs421/Create_Homelab_Project/assets/152476259/5ee74ed6-7df7-484a-a0f5-643734b9417f)

Back in the **Wazuh** dashboard, wait for a few moments and refresh the page to see the enrolled agent.

![wazuh dashboard agent](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ff50c929-4685-4417-9a3a-49c0f5aff547)


# Part 4: Generate Telemetry & Ingest into Wazuh

### Windows 10 Telemetry
We will be configuring the .conf file for **Wazuh** to generate telemetry and use **Mimikatz** as the sample attack .

In your Windows 10 VM, go to **C:\Program Files (x86)\ossec-agent** and look for the **ossec.conf** file. We will be configuring this config file to generate telemetry. 

Before opening the file, make a backup by copy-pasting the .conf file and renaming it **ossec-backup**.

You can open the .conf file in **Notepad** with **Administrative Privileges**.

We will be adding syntaxes right below this line group:

![log analysis](https://github.com/cs421/Create_Homelab_Project/assets/152476259/407326df-356b-4dad-ad94-e3bfba5eb77b)

You can also copy the syntax above and build rules from it. Paste the template just below the former syntax.

First, we will build a rule that will ingest **Sysmon** logs into **Wazuh**. We will be needing **Sysmon's** channel name for the `<location>` tag. **Sysmon's** channel name can be located via **Event Viewer**:

Open up **Event Viewer**, expand **Applications and Services Logs** -> **Microsoft** -> **Windows**, then scroll down to the **Sysmon** folder and click it. Right click on **Operational** and select **Properties**. 

![event viewer 1 1](https://github.com/cs421/Create_Homelab_Project/assets/152476259/dd48cf93-7dd9-4177-8e19-58af2c292a16)

![sysmon properties](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ac09eccd-bd1e-4e08-8cc5-37698eacf399)

![sysmon full name](https://github.com/cs421/Create_Homelab_Project/assets/152476259/564a4d29-f1bb-4b41-b10e-b88baf764a4c)

Copy the **Full Name** and replace the `Application` value in the syntax.

![sysmon channel name](https://github.com/cs421/Create_Homelab_Project/assets/152476259/704eba55-0b25-43e0-bd56-1a48af8a6c78)

For the sake of ingestion, we can remove the **Application**, **Security**, and **System** syntax and just retain the **Sysmon** syntax. This means that application, security, and system logs will not be forwarded into **Wazuh**. This is fine because we only want to observe **Sysmon** logs.

![remove app security syntax](https://github.com/cs421/Create_Homelab_Project/assets/152476259/7e9c2e50-d991-40c4-846d-597cf3586c41)

![remove system syntax.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/remove%20system%20syntax.png)

Save the file and exit.

Now we need to restart **Wazuh** by going to Services -> **Wazuh** and clicking Restart.

![restart wazuh.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/restart%20wazuh.png)

**Note:** Every time you change settings in the config file, you **MUST** restart the **Wazuh** service.

We can now go back to the **Wazuh** dashboard and search for "sysmon" events. It may take some time for events to show up.

![wazuh sysmon search.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20sysmon%20search.png)

### Downloading Mimikatz

**Mimikatz** is a credential harvesting application used by pentesters and hackers to extract credentials from a machine.

Before downloading **Mimikatz** we need to exclude our *Downloads* folder from Windows Security's scanning scope. 

Go to **Windows Security,** click on **Virus & threat protection** -> **Manage settings**, scroll down to the **Exclusions** section, and click **Add or remove exclusions**.

![windows security.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/windows%20security.png)

![virus threat protection.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virus%20threat%20protection.png)

![manage settings.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/manage%20settings.png)

![exclusions.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/exclusions.png)

Click on **Add an exclusion** and choose "Folder", then browse to your *Downloads* folder and select it.
![exclude download folder.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/exclude%20download%20folder.png)

![select download folder.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/select%20download%20folder.png)

Your *Downloads* folder should appear on the list.

![download folder list.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/download%20folder%20list.png)

Using **Google Chrome** is recommended when downloading **Mimikatz** because you can easily disable the browser protection from the security settings.

![chrome settings.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/chrome%20settings.png)

![chrome no protection.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/chrome%20no%20protection.png)

**Mimikatz** download link: [https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919](https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919)

Right click on the **.zip** file and **Save As**, and save it to the *Downloads* folder.

![download mimikatz.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/download%20mimikatz.png)

Go to your *Downloads* folder, right click on the zip file and select "Extract all"

![extract mimikatz.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/extract%20mimikatz.png)

Double click on the newly extracted folder and go inside the **x64** folder.

Open up **Powershell** with **admin** privileges and **cd** into the **x64** folder.

![powershell mimikatz folder.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/powershell%20mimikatz%20folder.png)

Run `.\mimikatz.exe`.

![mimikatz exe.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/mimikatz%20exe.png)

Going back to the **Wazuh** dashboard, if you search for "mimikatz" events, you may not get anything yet. 

![wazuh mimikatz alert 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20mimikatz%20alert%201.png)

This is due to the fact that **Sysmon** or **Wazuh** rules may not be triggering alerts, because, by default, **Wazuh** will only log events whenever a rule or alert is triggered.

To fix this, we will need to configure the **ossec.conf** file in our **Wazuh manager** so that it will log everything, and/or create a rule or rules specifying a particular event so that **Wazuh** can log that event whenever that rule is triggered.

### Configuring the Wazuh manager config file

Go to the **Wazuh** manager terminal by accessing it via the Digital Ocean Droplet console or establishing an **ssh** connection.

The **ossec.conf** file is located at `/var/ossec/etc/ossec.conf`.

First, we'll create a backup copy of the config file by typing `cp /var/ossec/etc/ossec.conf ~/ossec-backup.conf`
![wazuh linux ossec backup.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20linux%20ossec%20backup.png)

Now we'll edit the .conf file in **Nano**: `nano /var/ossec/etc/ossec.conf`

Scroll down to the `<logall>` and `<logall_json>` options and change the value from 'no' to 'yes'

![wazuh manager nano.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20manager%20nano.png)

Save and exit the file.

Now we need to restart the **Wazuh** service by `systemctl restart wazuh-manager.service`.

By then, **Wazuh** will now log all events and put it in a file called "**archives**", which is located at `/var/ossec/logs/archives`.

![wazuh archives 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20archives%201.png)

In order for **Wazuh** to start ingesting the logs, we need to configure our **filebeat** file. **Filebeat** is a log forwarder used by **Elasticsearch**. 

We can edit the **filebeat** config file in **nano** by `nano /etc/filebeat/filebeat.yml`.

Scroll down to the `filebeat.modules` option, and change the `enabled` value under `archives` from false to `true`.

![filebeat true.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/filebeat%20true.png)

Save the file and exit.

Restart the **filebeat** service by `systemctl restart filebeat`.


### Creating a new Wazuh index

In our **Wazuh** dashboard, click on the **3-line icon**, click on **Stack Management**, and select **Index Patterns**.
![wazuh stack management.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20stack%20management.png)

![index patterns.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/index%20patterns.png)

Inside **Index patterns**, we will originally see three patterns.

![index pattern list.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/index%20pattern%20list.png)

We will need to create a new index pattern for our **archives**. Click on the **Create index pattern** and name the new pattern as `wazuh-archives-**`

![create new index.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/create%20new%20index.png)

![wazuh archives name.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20archives%20name.png)

Click on **Next**, then select `timestamp` for the **Time field**, then click **Create index pattern**.

![timestamp time field.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/timestamp%20time%20field.png)

To make `wazuh-archives-**` our default index pattern, click on the **3-line icon** again and select **Discover**.

![discover.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/discover.png)

Then click the dropdown button and select `wazuh-archives-**`.

![wazuh archive default.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20archive%20default.png)

It will take time before events start to flow in, but eventually they will come.

You can also re-run **Mimikatz** in the Windows 10 client to see if any alerts appear in **Wazuh**.

In the **Mimikatz** terminal, exit from the first run by typing `exit` and running it again.

![mimikatz rerun.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/mimikatz%20rerun.png)

Back in the **Wazuh** dashboard, search for "mimikatz" events again, and a couple of alerts should appear. We should focus more on the alert having an **Event ID** of "1", because it is the event ID for **process creation**. 

![mimikatz alert.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/mimikatz%20alert.png)

Expanding the alert for more details, we can scroll down and see a field called `originalFileName`. We can use this field to create our alert for **Mimikatz**. This field is more reliable to use than the `image` field, because attackers can simply rename the file and it will reflect on that field, and they can bypass the alert, whereas the `originalFileName` field keeps the original name of that file, regardless of any renaming process.

![orignalfilename mimikatz.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/orignalfilename%20mimikatz.png)


### Creating Rules in Wazuh

In the **Wazuh** dashboard, click on the drop-down button, click on **Management**, and select **Rules**.

![wazuh create rules.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20create%20rules.png)

Click on **Manage Rules Files**.

![manage rules files.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/manage%20rules%20files.png)

Search for "sysmon" and look for **0800-sysmon_id_1.xml**. Preview the file by clicking on the **eye** icon

![sysmon event id 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/sysmon%20event%20id%201.png)

We can copy the first rule and use it as our template.
![sysmon copy rule.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/sysmon%20copy%20rule.png)

Go back from the sysmon xml rule and click **Custom rules**.

![custom rules.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/custom%20rules.png)

Inside, we can see the `local_rules.xml` file. We can edit this by clicking on the **pencil** icon.

![local rules.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/local%20rules.png)

We can paste the rule we copied earlier below the already existing rule inside **local_rules**.

**Note:** We must ensure and follow proper identations and naming of values in order for the rules to be applied successfully.

- By default, rule IDs start from **100000**, so we'll change the `rule id` to **100002**, since there is already another rule above us.
- The `level`'s max range is **15**, so we'll just set it to max.
- For the `field name`, we'll set it to `win.eventdata.originalFileName`, then change `\\(c|w)script\.exe` to `mimikatz\.exe` inside the `type` field.
- Remove the `<options>` field, then change the `<description>` to `Mimikatz Usage Detected`.
- Change the **mirtre id** to `T1003`,  (**credential dumping**), which is what **Mimikatz** is known to do.

![create mimikatz rule.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/create%20mimikatz%20rule.png)

Save the rule and restart the manager.

Before we run **Mimikatz** again, we can try renaming the .exe file into something different.
![rename mimikatz.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/rename%20mimikatz.png)

Now we'll run **Mimikatz** in powershell once more.

![verylegitprogram.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/verylegitprogram.png)

Refresh the **Wazuh** dashboard, then go to **Security Events** to check if any alert has been triggered.

![mimikatz alert triggered.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/mimikatz%20alert%20triggered.png)

![mimikatz original name.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/mimikatz%20original%20name.png)

Even if we renamed **Mimikatz** into something else, the alert still triggered because we used the `originalFileName` value.


# Part 5: Connect Suffle (SOAR), Send Alert to TheHive, Send to SOC Analyst via Email

### Configuring Shuffle
Head over to the **Shuffle** site (https://shuffler.io) and create an account

Click on **Workflows**

![shuffle workflow.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20workflow.png)

Click on **New Workflow** to create a new workflow

![shuffle new workflow.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20new%20workflow.png)

Name the workflow as "**SOC Automation Project**" and type any description that you like. For the **Usecases**, you can select any in the list. Click **Done** when you're ready.

![shuffle name workflow.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20name%20workflow.png)

This will be your new dashboard

![shuffle dashboard.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20dashboard.png)

Navigate to **Triggers** on the bottom left corner, then drag and drop the **Webhook** app into the dashboard.
![shuffle add webhook.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20add%20webhook.png)

Name the webhook as "**Wazuh-Alerts**", then copy the **Webhook URI**. We will need this URI to put into the **ossec.conf** file in our **Wazuh** manager.
![shuffle name webhook.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20name%20webhook.png)

Click on the **Change Me** icon, select the **Repeat back to me** option in the **Find Actions** tab, and on the **Call** tab, click on the plus icon and select **Execution Argument**.
![changeme settings.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/changeme%20settings.png)

Save the changes.
![changeme save.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/changeme%20save.png)

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

![shuffle integration.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20integration.png)

Replace `YOUR_SHUFFLE_URL` with the url that we copied from **Shuffle** earlier. Add a space (  ) between the url and the `</hook_url>` closing tag to avoid the tag becoming a part of the url. This is indicated by having the url highlighted, but not the `</hook_url>` tag.

The `<rule_id>` is the rule ID for **Mimikatz** that we configured earlier in the **Wazuh** custom rules.

Save and exit the editor.

Restart the wazuh manager service by `systemctl restart wazuh-manager.service` and check the status by `systemctl status wazuh-manager.service`


#### Testing the integration

In your Windows 10 VM, re-run **Mimikatz** to generate telemetry.

In the **Shuffle** dashboard, click on the **Wazuh-Alerts** webhook and click **Start**. Then click on the running man icon at the bottom to show executions.

![shuffle test telemetry.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20test%20telemetry.png)

This should show results regarding the telemetry. Click on the result and expand the **Execution Argument** tab.

![shuffle telemetry result.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20telemetry%20result.png)

This shows us the information that is generated from **Wazuh**.
![shuffle info from wazuh.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20info%20from%20wazuh.png)

Before moving on with the **IOC Enrichment** setup, we'll review our current objectives for the workflow:
1. **Mimikatz** alert sent to **Shuffle**
2. **Shuffle** receives **Mimikatz** alert and extracts **SHA256** hash from file
3. Check reputation score with **VirustTotal**
4. Send details to **TheHive** to create alert
5. Send email to **SOC Analyst** to begin investigation


### IOC Enrichment

![hash value.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/hash%20value.png)


The file's return value for the **hashes** is appended by their hash type (e.g., **SHA1, MD5**). We will need to parse out the hash value itself to be sent to VirusTotal.


Click on the **Change Me** icon, and search for "**Regex capture group**" in the **Find Actions** tab, and select it. You can also change the name of the icon to "**SHA256_Regex**".

![regex capture group.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/regex%20capture%20group.png)_

In the **Input data**, select **Execution Argument** and find the **hashes** parameter. You can check the values of a parameter by hovering over it and looking at the details that appear beside it.

![input data.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/input%20data.png)

![execution argument hashes.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/execution%20argument%20hashes.png)

For the **Regex** field, we can utilize **ChatGPT** to help us create the formula for the hashes. We'll type a prompt to create a regex to parse our file's sha256 value. Copy the hash details of the file earlier and paste it in the prompt as seen below.

![regex field.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/regex%20field.png)

![chatgpt hash prompt.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/chatgpt%20hash%20prompt.png)

We can now copy the regex that **ChatGPT** has created after it has finished generating. 

![chatgpt finished regex.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/chatgpt%20finished%20regex.png)

Paste the regex into the field and save the workflow.

![regex pasted.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/regex%20pasted.png)

Click on the running person icon to show executions, then click the refresh button on top to re-run executions.

![refresh workflow.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/refresh%20workflow.png)

Expanding on the results will show us the parsed SHA256 hash of the file.

![regex parse result.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/regex%20parse%20result.png)


#### Setting up VirusTotal

Go to the **VirusTotal** (https://www.virustotal.com/) website and create an account 

After signing up, request for your API key and copy it.

![virustotal api1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20api1.png)

![virustotal api2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20api2.png)


In the **Apps** tab, search for **VirustTotal** and click it to activate the app. Once it has been added in your instance, drag and drop it into the workflow. Save the workflow first. Wait for a couple of minutes for **VirusTotal** to finish loading, then refresh the page.

![virustotal app.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20app.png)

![virustotal drag and drop.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20drag%20and%20drop.png)


##### Checking VirusTotal report file path

Before we delve more into configuring **Virustotal** in the workflow, we must double check first its file path on reports, because it has become an issue to several users whenever they run the workflow with **VirusTotal**.

According to the **VirusTotal** API documentation (https://docs.virustotal.com/reference/file-info), the correct file path for getting a file report by hash is in `https://www.virustotal.com/api/v3/files/{id}`

![virustotal filepath.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20filepath.png)

We should take note of this file path.

Going back to the **Shuffle** dashboard, click on **Apps** on top of the page.

![shuffle go to apps.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20go%20to%20apps.png)

Click on the icon on the top right of **VirusTotal**.

![virustotal top right.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20top%20right.png)

Click the **FORK** button to edit the files.

![virustotal fork.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20fork.png)

Click on **Get a hash report**, then make sure that the **URL path/Curl statement** value is `/files/{id}`. Then click on **Submit** and **SAVE**.

![virustotal edit file 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20edit%20file%201.png)

![virustotal edit file 2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20edit%20file%202.png)

![virustotal edit file 3.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20edit%20file%203.png)


#### Configuring the VirusTotal app

Click on the **VirusTotal** icon to change its settings. You can change the name to just "Virustotal".

In the **Find Actions** option, search for "hash" and select **Get a hash report.**
![virustotal get hash report.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20get%20hash%20report.png)

Click on **AUTHENTICATE VIRUSTOTAL V3** for us to authenticate using our API key.
![authenticate virustotal.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/authenticate%20virustotal.png)

You can paste your API key here, then click **Submit**.
![authenticate virustotal2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/authenticate%20virustotal2.png)


In the **Hash** field, click on the plus icon, select **SHA256_Regex**, then select **list**.

![hash field regex.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/hash%20field%20regex.png)

![hash field regex2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/hash%20field%20regex2.png)

Save your workflow after. Click on the running person icon to show executions, then select the most recent **Wazuh** run. Then click on the refresh icon to re-run the workflow.

![virustotal rerun.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20rerun.png)

![virustotal rerun2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20rerun2.png)

After re-running the workflow, expand the **VirusTotal** results.

![virustotal result.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20result.png)

For some reason, the result **status** shows the **404** code, which means that the file or path cannot be found. The OK status should be **200**. 

According to **MyDFIR**, this is a technical or network issue on **Shuffler's** side, so we cannot do anything about it currently. But for now, we already know how to parse the file hash and send to **VirusTotal** to get a hash report and enrich **IOCs**.

![virustotal 404.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/virustotal%20404.png)

The next step is to send the details to **TheHive** so that it can create an alert for case management.

### Setting up TheHive

In the **Apps** tab, search for "thehive", then click on the app to activate it.

![thehive app.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20app.png)

Drag and drop **TheHive** app into the dashboard, save your workflow, wait for a couple of minutes for the app to finish loading its features, then refresh the page.

![thehive drag and drop.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20drag%20and%20drop.png)

Go to **TheHive** web interface located at `THEHIVE_VM_IP:9000`, then log in with the default credentials of `admin@thehive.local` and the password `secret`.


#### Creating a new organisation

By default, we will only see one organisation on the dashboard named "**admin**". We will need to create a new user for ourselves. 

Click on the **plus** icon on the top left to add an organisation. Name it however you want, then put "SOC Automation Project" in the **Description** section. Click **Confirm** when ready.
![thehive add organization.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20add%20organization.png)

We can now see two organisations, the default **admin** and our newly created one. Click the latest organisation to view its users.

![thehive organizations.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20organizations.png)


#### Creating users

Currently, there are no users yet, so we should create one. Click on the **plus** icon.

![thehive add user.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20add%20user.png)

We will leave the **Type** as "Normal", then choose your preferred login with the suffix "`@test.com`". Type your preferred name, then in the **Profile** tab, choose **analyst**. Then click **"Save and add another"**.

![thehive save add another.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20save%20add%20another.png)

For the other user, we will select the **Type** as "Service". For the **Login**, we will use `shuffle@test.com` and name it "**SOAR**". We will choose "analyst" for the **Profile**, and click **Confirm**.

**NOTE:** In a real-world scenario, we should be assigning profiles for each user with the **principles of least privilege** in mind. 
![thehive add shuffle user.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20add%20shuffle%20user.png)

Next, we will create a password for our main user. In the dashboard, click on **PREVIEW** when you hover over your user link.
Click on **Edit password** to create your password, then click **Confirm**. 

![thehive preview user.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20preview%20user.png)

![thehive set password.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20set%20password.png)

For our **SOAR** user, click on **Create** on the **API Key** section and copy it. We will need this to authenticate with **Shuffle**. 

Once the password is set and the API key is created, log out of the admin account, and log in with your user account.

These are the default dashboards for the cases and alerts inside the analyst page.

![thehive cases.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20cases.png)

![thehive alerts.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20alerts.png)


### Authenticating TheHive with Shuffle

Go back to the **Shuffle** dashboard. Click on **TheHive** app and select **AUTHENTICATE THEHIVE 5**.

Paste your **SOAR** API key in the field, and change the url into your **TheHive**'s public ip + port 9000. Click **Submit** when finished. Make sure to save your workflow.

![authenticate thehive.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/authenticate%20thehive.png)

Connect **VirusTotal** to **TheHive**. Then click on **TheHive**, and on the **Find Actions** tab, select **Create Alert**.

![thehive settings 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20settings%201.png)

Scroll down to the **Date** section. Select **Execution Argument**, then select the **utcTime** value under **eventdata**.

![thehive set date.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20set%20date.png)

![thehive utctime.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20utctime.png)

For the **Description**, type the following message and values:

```
Mimikatz detected on host: "Execution Argument -> system -> computer`"
from user: "Execution Argument -> eventdata -> user"
```

![thehive computer value.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20computer%20value.png)

![thehive user value.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20user%20value.png)

Scroll down and set the following values:
- **Flag** = `false`
- **Pap** = `2`
	- **Pap** is short for **permissible actions protocol**, which means the level of exposure of information.
- **Severity** = `2`
- **Source** = `Wazuh`
- **Sourceref** = `"Rule: 100002"`
- **Status** = `New`

![thehive settings 2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20settings%202.png)

For the **Summary**, type the following message and values:
```
Mimikatz detected on host: `Execution Argument -> system -> computer` 
and the Process ID is: 'Execution Argument -> eventdata -> processId`
and the Command Line is: 'Execution Argument -> eventdata -> commandLine`
```

![thehive computer value.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20computer%20value.png)

![thehive process id.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20process%20id.png)

![thehive command line.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20command%20line.png)

For the **Tags** = ["T1003"]

![thehive tags.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20tags.png)

For the **Title**, we can parent it to the alert title.

![thehive alert title.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20alert%20title.png)

For the **Tlp** = `2`
- **Tlp** is short for **Traffic light protocol**, which is the level of confidentiality of information

For **Type** = `Internal`
![thehive settings 3.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20settings%203.png)

Save your workflow after this.

Before testing the workflow, we will need to configure our cloud firewall to allow inbound connections in port 9000. 
Go back to the **DigitalOcean** dashboard and navigate to the firewall options.

![firewall 9000_1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/firewall%209000_1.png)

Create a rule that allows inbound **TCP** connections in port **9000**. We can remove IPv6 connections, but retain **All IPv4**. 

![firewall 9000_2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/firewall%209000_2.png)

Save the settings then go back to the **Shuffle** dashboard.

In the **Shuffle** dashboard, click on the running person icon to show executions, then refresh the run.

![thehive test run.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20test%20run.png)

We can now see that **TheHive** has successfully created an alert.

![thehive result.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20result.png)

To confirm this, we'll go back to our **TheHive** dashboard.

Sure enough, the alert is now present in the dashboard. Expanding on the alert will show us the details that we inputted back in **Shuffle**.

![thehive mimikatz alert.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20mimikatz%20alert.png)

![thehive mimikatz alert expand.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20mimikatz%20alert%20expand.png)

The next step is to send an email to the analyst with the information gathered. 


### Sending Alert to Analyst via Email

In the **Apps** tab, search for "e-mail", then click on the **Email** app to activate it.

![thehive email.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20email.png)

Drag and drop the **Email** app into the dashboard, then connect **VirusTotal** to it.

![thehive virustotal connect email.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20virustotal%20connect%20email.png)

Click on the **Email** app inside the dashboard to access its settings.
For the **Recipients**, it is best to provide a temporary or disposable email address for this lab.

For the Subject, we can type "**Mimikatz Detected!**"

For the **Body**, we can type the following message and values:
```
Time: `Execution Argument -> eventdata -> utcTime`
Title: 'Execution Argument -> title'
Host: 'Execution Argument -> system -> computer'
```

![thehive utctime.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20utctime.png)

![thehive alert title.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20alert%20title.png)

![thehive computer value.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/thehive%20computer%20value.png)

Save your workflow after.

Click on the running person icon again to show executions, then refresh the run. 

We can now see that **Shuffle** has successfully sent an email to the analyst.
![shuffle email sent.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20email%20sent.png)

Going to our inbox, we see the message sent by **Shuffle**, alerting the analyst of the detection.

![inbox email received.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/inbox%20email%20received.png)


# Part 6: Ubuntu Client and Responsive Action

### Creating an Ubuntu machine in DigitalOcean

![create droplet 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/create%20droplet%201.png)

![ubuntu droplet settings 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20droplet%20settings%201.png)

![ubuntu droplet settings 2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20droplet%20settings%202.png)

![ubuntu droplet settings 3.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20droplet%20settings%203.png)

![ubuntu droplet settings 4.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20droplet%20settings%204.png)

![ubuntu agent dashboard.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20agent%20dashboard.png)

![ubuntu new firewall.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20new%20firewall.png)


### Installing Wazuh agent in Ubuntu

Access the Ubuntu machine by either the **Droplet** console in **DigitalOcean** or via ssh, depending on which one is faster for you.

Then run the following commands to install the **Wazuh** agent.

`sudo apt-get update && apt-get upgrade`

`curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg`

![ubuntu instal gpg key.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20instal%20gpg%20key.png)

`echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list`

![ubuntu add repository.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20add%20repository.png)

`apt-get update`


`WAZUH_MANAGER="10.0.0.2" apt-get install wazuh-agent`

![ubuntu wazuh finish install cli.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20wazuh%20finish%20install%20cli.png)

`systemctl daemon-reload`
`systemctl enable wazuh-agent`
`systemctl start wazuh-agent`

`sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list`
`apt-get update`

After installing the **Wazuh** agent, we will go back to the **Wazuh** web interface to see the Ubuntu machine registered.

In the **Security Events,** we will also see a flood of brute-force ssh attempts in our machine. 

![wazuh ubuntu agent success.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20ubuntu%20agent%20success.png)

![ubuntu ssh attempts.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ubuntu%20ssh%20attempts.png)


### Block SSH attempts on Ubuntu agent

We will need the **API user** credentials that we took note in the earlier steps inside our **Wazuh** manager. The **.txt** file is located at `wazuh-install-files/wazuh-passwords.txt`.

![cat wazuh password.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/cat%20wazuh%20password.png)

![wazuh api user 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20api%20user%201.png)

Back in our **Shuffle** dashboard, we will disconnect our current workflow first and rebuild it with the responsive action workflow included. We need to stop the **Wazuh-Alerts** app first, then detach the branches.

![shuffle stop wazuh.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20stop%20wazuh.png)

![shuffle delete branch.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20delete%20branch.png)

In the **Apps** tab, drag and drop the **Http** app and rename it as "**Get-API**". Select **Curl** in the **Find Actions** tab.

![http drag and drop.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/http%20drag%20and%20drop.png)

![http rename get api.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/http%20rename%20get%20api.png)

Select **Expand window** in the **Statement** tab and input the following command:
`curl -u USER:PASSWORD -k -X GET "https://WAZUH-IP:55000/security/user/authenticate?raw=true"`

Be sure to replace the values of **USER**, **PASSWORD**, and the **WAZUH-IP** in the command. The user and password values refer to our **wazuh api user** credentials. 

Click **Submit** when finished and save your workflow.

![curl expand.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/curl%20expand.png)

![curl expand command line.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/curl%20expand%20command%20line.png)


In the **Apps** tab, search for "wazuh" and click on the app to activate it.

![shuffle wazuh app.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20wazuh%20app.png)

Drag and drop the **Wazuh** app into the dashboard

![wazuh drag and drop.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20drag%20and%20drop.png)

We can partially rebuild our workflow to test the responsive capability without the presence of user input yet.

First, we should run the workflow with only the **Wazuh Alerts** trigger and **Get-API** app connected to collect report details.

![get report detail.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/get%20report%20detail.png)

![get api report detail.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/get%20api%20report%20detail.png)

These details will be important in feeding into **Virustotal** and **Wazuh** later on.

![srcip.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/srcip.png)

Also, take note of the **srcip** under the data section, we will be pinging it later. 


#### Reconfiguring Virustotal Action

Click on the **Virustotal** app, and in the **Find Actions** section, select "**Get an IP address report**".

For the **Ip** section, input `$exec.all_fields.agent.ip`.

![reconfigure virustotal.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/reconfigure%20virustotal.png)

![agent ip.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/agent%20ip.png)


### Restructuring the workflow

Click on the **Wazuh-Alerts** app and start it. Then we can connect the following apps: **Wazuh-Alerts** -> **Get-API** -> **Virustotal** -> **Wazuh**.

![shuffle partial rebuild.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20partial%20rebuild.png)

Click on the **Wazuh** app to edit its details. We will leave the **Find Actions** tab on "Run command", then on the **Apikey** tab, click the plus icon and select **Get-API**.

![wazuh shuffle details 1.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20shuffle%20details%201.png)

Input your **Wazuh** public IP on the **localhost** value inside the **Url** tab.
![wazuh shuffle details 2.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20shuffle%20details%202.png)

For the **Agents list** tab, we will need to identify our machine's **Agent ID**. One way to view it is to go to the **Agents** dashboard in the **Wazuh** interface and looking at the **ID** row on the machines. 

![wazuh agent id.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20agent%20id.png)

We can also select its values by clicking the plus icon, then "**Execution Argument** -> **agent** -> **id**".

![shuffle select agent id.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20select%20agent%20id.png)

Select the value for **Wait for complete** to "true".

![wait for complete true.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wait%20for%20complete%20true.png)

Before moving on with the rest of the settings, we will need to access our **Wazuh** manager console to configure the active response settings.

Open the **ossec.conf** file with **nano** located at `/var/ossec/etc/ossec.conf`

Search for "active response" by `ctrl+w` and scroll down to the listed commands under it. The command that we will be using is `firewall-drop`

![conf file active response.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/conf%20file%20active%20response.png)

`firewall-drop` will modify the IP tables of the Ubuntu machine and drop all traffic inbound the machine. 
We will make an active response tag and make sure that the command reflects the command name we want to use. 

Scroll down until we see the `<active-reponse>` tag. We will modify this to use the `firewall-drop` command.

![active response tag.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/active%20response%20tag.png)

The final tag should look like this:

![active response modified.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/active%20response%20modified.png)

The `local` location means that the active response will be implemented by wherever the alert was triggered from. In our case, the local location is our Ubuntu machine.

Save the file and exit. 

We will restart the wazuh manager through `systemctl restart wazuh-manager`


### Testing Active Response

Before moving on, one important concept about working with API's is that the command name appends the timeout to the name.

Example of this is the `firewall-drop` command. If we want to use this command via API, we must append the timeout that we set earlier to this command. Since we set the timeout to "**no**", the value is **0**, hence the command will be `firewall-drop0`

One way to see the name and test to see if the script is active is to use an **agent control binary**, which is located at `/var/ossec/bin`.

![agent control location.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/agent%20control%20location.png)

Running `agent_control` without options will show us this list:

![agent control options.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/agent%20control%20options.png)

We can use `-L` to list available active responses.

![agent control response name.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/agent%20control%20response%20name.png)

This shows us the response name of the active reponse, which is `firewall-drop`, along with the timeout value appended to it. 


#### Pinging Google DNS on Ubuntu Machine

To test our active reponse action, we can ping **Google's** DNS (8.8.8.8) on our Ubuntu machine.

![ping google.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ping%20google.png)

Then, while the ping is running, we can execute the following command in our **Wazuh** manager console: 
`./agent_control -b 8.8.8.8 -f firewall-drop0 -u 002`

Going back to our Ubuntu machine, we see that it has stopped pinging. Looking at `iptables --list` shows us that it drops everything inbound from Google's DNS.

![ping no response.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ping%20no%20response.png)

![iptables list.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/iptables%20list.png)


We can check if the active response is working by looking at the `active-reponses.log` located at `/var/ossec/logs`

![active response log location.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/active%20response%20log%20location.png)

![active response log.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/active%20response%20log.png)


Back in our **Shuffle dashboard**, enter the following values for the **Alert** and **Command** tabs:
- For **Alert**: `{"data":{"srcip":"8.8.8.8"}}`
- For **Command**: `firewall-drop0`

![wazuh shuffle details 3.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20shuffle%20details%203.png)

Save your workflow.

Before running executions in **Shuffle**, we'll go back to our Ubuntu machine to ping Google's DNS again. We can type `iptables --flush` to flush out our earlier ping session and begin a new one.

Ping Google's DNS at **8.8.8.8** and, then switch back to the **Shuffle** dashboard, click the running person icon to show executions and refresh the run. 


Checking at the result, it shows the affected items (agent id: **002**), and checking the IP tables on the Ubuntu machine shows Google's DNS being dropped.

![wazuh active response result.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20active%20response%20result.png)

![wazuh active response ip table result.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20active%20response%20ip%20table%20result.png)


### Enabling User Input on alerts

In this step, we will set up a user input to send an email to the analyst with the alert info provided. If the analyst chooses to say yes and block the IP, **Wazuh** will then automatically instruct the Ubuntu machine to block said IP.

In the **Triggers** tab, drag and drop the **User Input** trigger into the dashboard.

![user input trigger.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/user%20input%20trigger.png)

We can then remove the **Wazuh** app from the flow for the time being and replace it with the **User input** trigger.

![wazuh replace user input.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20replace%20user%20input.png)

Click the **User input** trigger to change its settings.

In the **Information** tab, type the following message and command:

```
Do you want to block this source IP: $exec.all_fields.data.srcip ?
```

In the **Input options**, tick the **Email** box and input your analyst email address, preferably a temporary one.

![user input settings.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/user%20input%20settings.png)

We will now connect the **Wazuh** app back after the **User input** trigger.
![wazuh connect back.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20connect%20back.png)

In the **Alert** tab, remove **8.8.8.8** to replace it with another IP via `$exec.all_fields.data.srcip`.

![wazuh source ip.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/wazuh%20source%20ip.png)

Save your workflow after this.


### Testing the final workflow

In your Ubuntu machine, we will start pinging the source IP that we took note of earlier.

![ping ip.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ping%20ip.png)

In our **Shuffle** dashboard, re-rerun the the execution with all nodes connected.

![shuffle final run.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20final%20run.png)

After running the execution, you should be able to receive an email from **Shuffle** requiring user input.

![userinput email.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/userinput%20email.png)

Clicking the **TRUE** link will forward you to a confirmation page telling us that the action is implemented.

![shuffle action done.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/shuffle%20action%20done.png)

Going back to the Ubuntu machine, we can see that the ping has stopped and the IP connection was dropped.

![ip dropped.png](https://github.com/cs421/Create_Homelab_Project/blob/main/SOC%20Automated%20Project/attachments/ip%20dropped.png)
