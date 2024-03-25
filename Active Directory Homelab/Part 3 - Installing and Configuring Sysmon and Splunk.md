## 1. Configuring NAT Network
We will need to configure our network in VirtualBox to have NAT Network enabled in our machines so they can talk to each other.

- In VirtualBox, click on the right side icon of the **Tools** bar, then navigate to the "**NAT Network**" tab and click "Create."
![[nat network 1.png]]
![[nat network 2.png]]

- Double click on the newly created "NatNetwork" to configure it. For the name, it can be anything you want.
- For the **IPv4 Prefix**, we will be using the network that we initialized back in the diagram, which is **192.168.10.0/24**. Click "Apply" when finished.
![[nat network 3.png]]
We will be changing the network adapter settings in our virtual machines to use the **NatNetwork**.

- Click on **Settings** in the virtual machine, and navigate to the **Network** tab.
- Select "NAT Network" in the **Attached to** dropdown, and double check the name of the network. Click OK when finished.
![[nat network 4.png]]![[nat network 5.png]]

- Repeat the process for the remaining three machines.
![[nat network 6.png]]![[nat network 7.png]]
![[nat network 8.png]]

## 2. Configuring Splunk Server
### 2.1 Configuring IP Address
In our Splunk server, we will first configure our IP address according to the assigned address in our diagram.

- We can check the IP address by `ip a`
![[splunk nat 1.png]]
The current IP address for the Splunk server is **192.168.10.4**, which is not the address that was assigned in the diagram, so we will need to change this.

- `sudo nano /etc/netplan/00-installer-config.yaml` to configure the netplan file.
- Change the value for **dhcp4** to **no**
![[splunk nat 2.png]]

- Press Enter to add a new line, then press TAB three times for the margin.
- Type **addresses:**, then, in brackets, input the static IP **192.168.10.10** followed by the **/24** subnet.
![[splunk nat 3.png]]
 
- Press Enter to add a new line, then press TAB three times for the margin.
- Type **nameservers:**, then press Enter for a new line. 
- For the next margin, press TAB five times. Type **addresses:**, then input **[8.8.8.8]** for the DNS server. This is Google's DNS server.
![[splunk nat 4.png]]

- Press Enter to add a new line, then press TAB three times for the margin.
- Type **routes:**, then press Enter
- For the next margin, press TAB five times, then type "**- to:**", then input "**default**" for the value.
- Press Enter to add a new line, then press TAB six times for the margin.
- Type **via:**, then input **192.168.10.1** for the value. This is the gateway address.
![[splunk nat 5.png]]

Save the file by pressing **CTRL+x** and **Y**. 
![[splunk nat 6.png]]
- Press Enter to save to file and exit.
![[splunk nat 7.png]]

- To apply the settings, type `sudo netplan apply`. We may get some errors, but it's fine, this being only a homelab.
- Double check the IP address by `ip a`. We can see that the IP address has been changed accordingly.
![[splunk nat 8.png]]
- We can also ping Google to check our connectivity.
![[splunk nat 9.png]]

### 2.2 Installing Splunk Enterprise

- In your host machine, go to www.splunk.com and sign up for a free account.
![[splunk sign up.png]]

- After signing up, go to **Free Trials & Downloads** under the **Products** tab.
![[splunk download.png]]

- Click **Get My Free Trial** for the **Splunk Enterprise**.
![[splunk download 2.png]]
- Choose the **Linux** version, and the **.deb** file. Save it preferably in the directory where your virtual machines are located.
![[splunk download 3.png]]

- In our **Splunk** virtual machine, we will be installing guest add-ons so we can install **Splunk Enterprise** in it.
- Type `sudo apt-get instal virtualbox`, then press TAB to autocomplete.
- We will be installing **virtualbox-guest-additions-iso** and **virtualbox-guest-utils**. We'll proceed with **virtualbox-guest-additions-iso** first.
- Type `sudo apt-get install virtualbox-guest-additions-iso`
![[splunk addon 1.png]]
- Type "**y**" to proceed with the installation
![[splunk addon 2.png]]
- The install has been finished when you see this screen.
![[splunk addon 3.png]]

- Now we'll install **virtualbox-guest-utils**. Type `sudo apt-get install virtualbox-guest-utils`. This install will be much quicker than the previous one.
![[splunk addon 4.png]]

- To access the Splunk **.deb** file we downloaded earlier, go to Devices -> Shared Folders -> Shared Folders Settings.
![[splunk addon 5.png]]
- Click on the **Add folder** icon.
![[splunk addon 6.png]]
- For the **Folder Path**, choose the folder where the Splunk installer is saved. We can leave the **Folder Name** as is, or you can also name it however you want.
- Make sure that the **Read-only, Auto-mount**, and **Make Permanent** options are checked, then click OK.
![[splunk addon 7.png]]
- Click OK after this.
![[splunk addon 8.png]]

- Restart the virtual machine by typing `sudo reboot`.
![[splunk reboot 1.png]]
After booting up, log back in.

- We will need to add our user to the **vboxsf** group by typing `sudo adduser [username] vboxsf`.
![[splunk vboxsf.png]]

- Next, we will make a directory called "**share**" by typing `mkdir share`.
- We can check the directory status by `ls -la`.
![[splunk share 1.png]]

- We will now mount the shared folder to the **share** directory by typing `sudo mount -t vboxsf -o uid=1000,gid=1000 [shared folder] share/`
- The `[shared folder]` value is the folder name in the Shared Folders earlier.
![[splunk share 2.png]]
```ad-note
If you are having errors with the mount function, try logging out by typing `exit` then logging back in. Sometimes the `adduser` process will not take effect until you log out.
```
- To check if the mount is successful, type `ls -la` to list directories. The **share** directory should be highlighted if the mount is successful.
- We'll change directories into **share** by `cd share/`, then `ls -la` to show the contents of the shared folder, including the **Splunk** installer we downloaded.
![[splunk share 3.png]]
- To install **Splunk Enterprise**, type `sudo dpkg -i splunk-` then press TAB to autocomplete.
![[splunk deb.png]]
- When you see the **complete** text, it means that Splunk has been successfully installed.
- The directory for Splunk is located at */opt/splunk*. We'll need to change directories by `cd /opt/splunk`, then list the files by `ls -la`.
![[splunk opt.png]]
- We see that the user and group belongs to **splunk**, which is good because it limits the permissions for that directory.
- We can change to the user **splunk** by `sudo -u splunk bash`.
![[splunk user.png]]
- We are now acting as the user **splunk**.
- We'll change directories into **bin**, as all binaries that Splunk can use is in this directory.
![[splunk bin.png]]
- We will use the **./splunk** binary by `./splunk start`
- The license and agreement texts will appear, and we can exit this by pressing **q**, and then **y** to agree.
![[splunk terms.png]]
![[splunk term 2.png]]
- We will be presented to create an administrator name and password.
![[splunk create admin 1.png]]![[splunk create admin 2.png]]
- After the process is completed, we see now that we can access Splunk in the browser through http://splunk:8000.
- We can now exit from the user **splunk** by `exit`.
- Next, we will run a command that enables Splunk to run whenever the virtual machine boots.
- We'll `cd` back to the **bin** directory and type `sudo ./splunk enable boot-start -user splunk`. This will enable Splunk to run with the user **splunk** whenever the machine reboots.
![[splunk reboot 2.png]]
- Splunk is now up an running in the Ubuntu machine.

## 3. Installing Sysmon
### 3.1 On Target PC
#### 3.1.1 Renaming the PC
First, we'll rename our target PC into something more recognizable.

- In the taskbar, search for "This PC" and click on "Properties".
![[this pc.png]]

- Click on "Rename this PC" and name it "**target-PC**", then click "Next".
- Restart the machine and log back in.
![[rename pc 1.png]]
![[rename pc 2.png]]
![[rename pc 3.png]]
#### 3.1.2 Configuring Static IP

- Search "cmd" on the taskbar, and select **Command Prompt**
![[win10 ip.png]]
- In the Command Prompt, type `ipconfig` to view the IP details.
![[win10 ip 2.png]]
- Currently, our target machine's IP is **192.168.10.6**, but personally, this is a bit too close to our AD server's assigned IP of **192.168.10.7**. We can change this to avoid any potential collisions.
- In the taskbar, click on the network icon, and select "Network & Internet settings".
![[win10 ip 3.png]]
- Click on "Change adapter options".
![[win10 ip 4.png]]
- Right click on the "Ethernet" network adapter and click **Properties**.
![[win10 ip 5.png]]
- Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
![[win10 ip 6.png]]
- Select "Use the following IP address" and type in your desired IP address, which must be acceptable within our subnet of /24.
- For the **Default gateway**, type in **192.168.10.1**.
- For the DNS settings, type in **8.8.8.8** for Google's DNS server.
![[win10 ip 7.png]]
- Click OK and close the properties window.
- We can check our IP address by typing `ipconfig` again in the command prompt.
![[win10 ip 8.png]]

#### 3.1.3 Installing Splunk Universal Forwarder
We can now open up a web browser to check our connectivity with the Splunk server.
- Make sure that the Splunk virtual machine is powered on.
- On a web browser, type in `192.168.10.10:8000` to access the server.
```ad-important
Splunk listens on port 8000, so always make sure to specify the port to gain access to the server.
```
![[win10 splunk 1.png]]

To install **Splunk Universal Forwarder**, go to `www.splunk.com` and log in.
- Click on the **Products** tab, and select "Free trials & Downloads".
![[win10 splunk 2.png]]
- Scroll down to "Universal Forwarder" and click "Get My Free Download"
![[win10 splunk 3.png]]
- Choose the Windows 10 64-bit version and click "Download Now".
![[win10 splunk 4.png]]
- Read through the terms and agreement, check on the agree button, and click **Access program**.
![[win10 splunk 5.png]]
- After the download is done, go to the folder where the installer is downloaded and double click the .msi file.
- Check the box for the license agreement.
- Make sure to select "An on-premises Splunk Enterprise instance", then click "Next".
![[win 10 splunk 6.png]]
- For the username, type "admin", then leave the "Generate random password" button checked, and click "Next".
![[win10 splunk 6.png]]
- We'll skip the **Deployment Server** settings because we don't have one.
![[win10 splunk 7.png]]
- For the **Receiving Indexer** field, type the IP address for our Splunk server, which is **192.168.10.100**. The default receiving port for Splunk is **9997**, so we will just use this.
- Click **Install** to begin installation.
![[win10 splunk 8.png]]
- Click on **Finish** to finish installer
![[win 10 splunk finish.png]]
#### 3.1.4 Installing Sysmon

- Go to https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon and click **Download Sysmon**
![[win10 sysmon 1.png]]
- For the sysmon configuration file, we will be using olafhartong's file at https://github.com/olafhartong/sysmon-modular.
- Scroll down and select *sysmonconfig.xml*.
![[win10 sysmon 2.png]]
- Click on **Raw** to view in plaintext form.
![[win10 sysmon 3.png]]
- Right click and select "Save As", and save in the Downloads folder.
![[win10 sysmon 4.png]]
- Go to the Downloads folder, right click on the *Sysmon* zip file, and select **Extract All**.
![[win10 sysmon 5.png]]
![[win10 sysmon 6.png]]

- In the extracted file directory, click on the address bar and copy the file path.
![[win10 sysmon 7.png]]

- Open up powershell with administrative privileges.
![[win10 sysmon 8.png]]

- Type `cd [file path]` to change directory into the Sysmon extracted files
![[win10 sysmon 9.png]]
![[win10 sysmon 10.png]]
- Now we're inside the Sysmon's extracted files directory, type in `.\Sysmon64.exe -i ..\sysmonconfig.xml`.
	- The `-i` option indicates that we want to use a configuration file.
![[win10 sysmon 11.png]]

- Click "Agree" when the license agreement window pops up.
![[win10 sysmon 12.png]]

- This indicates that Sysmon has been installed.
![[win10 sysmon 13.png]]

#### 3.1.5 Configuring Data Forwarding to Splunk
We will be instructing Splunk Universal Forwarder on what we want the forwarder to send over to the Splunk Server.

- The configuration file that we will be referencing is called *inputs.conf*, in which the original file is located at `C://Program Files/SplunkUniversalForwarder/etc/system/default`
![[inputs conf 1.png]]
```ad-important
DO NOT edit *inputs.conf* file located at the default folder, instead, make a new *inputs.conf* file at the **local** folder. The main file in the default folder serves only as a backup in case there are issues with the new *.conf* file. 
```
- We will be making a new *.conf* file at the local folder.
![[inputs conf 2.png]]
- Since the local folder has administrative privileges, we will need to open up a Notepad file with admin privileges and paste the contents of *inputs.conf* into it.
![[inputs conf 3.png]]
- The *inputs.conf* content is available to copy at https://github.com/MyDFIR/Active-Directory-Project
- Copy and paste the contents into the notepad file.
```ad-note
Take note of the index value **endpoint**. This is the index that will receive data from our target PC into the Splunk server. Without it, Splunk will not be able to receive data and events from this machine. We will configure the indexes in the Splunk dashboard to include the **endpoint** index later on.
```
![[inputs conf 4.png]]
- Save the file as *inputs.conf* and choose the local folder as the location.
- Make sure to select "All Files" in the file type to save it other than as a *.txt* file.
![[inputs conf 5.png]]
![[inputs conf 6.png]]

- Next, go **Services** so we can restart the Splunk forwarder service and apply the changes.
![[services 1.png]]
- Scroll down and locate **SplunkForwarder**.
![[services 2.png]]
- Before we restart the service, we'll need to change its log on type from **NT Service** to **Local System**.
- Right click on the service and select **Properties**.
![[services 3.png]]
- Navigate to the **Log On** tab and select **Local System account** to log on as, and click **Apply**.
![[services 4.png]]
- Click OK to exit the window.
- We can now click **Restart** the service.
![[services 5.png]]
- Just click OK when you get this notification.
![[services 6.png]]
- Click **Start** to restart the service.
![[services 7.png]]
- Once **SplunkForwarder** has restarted, it will now log on as **Local System**.
![[services 8.png]]

Next, we will be logging in to our Splunk server web interface at `192.168.10.10:8000`

![[splunk server log in.png]]

We will configure the indexes to include the **endpoint** index so Splunk can receive data from the target PC.

- Go to the **Settings** tab and click **Indexes**.
![[splunk index 1.png]]
- These are the indexes that Splunk has deployed as default. We do not see the **endpoint** index needed to receive data from the target PC.
![[splunk index 2.png]]
- Click on **New Index** to create a new one.
![[splunk index 3.png]]
- In the **Index Name** field, type "endpoint" for the value. Click **Save** after.
![[splunk index 4.png]]

- We can now see our newly created index on the list.
![[splunk index 5.png]]

Next, we will enable Splunk to receive the data.
- Go to the **Settings** tab and click **Forwarding and receiving**.
![[splunk index 7.png]]
- Click on **Configure receiving**.
![[splunk index 8.png]]
- Click on **New Receiving Port**.
![[splunk index 9.png]]
- Type **9997** as the listening port. This is the default port that Splunk listens on. Click Save after.
![[splunk index 10.png]]

- We now see port **9997** as the listening port, and it is enabled.
![[splunk index 11.png]]

Splunk should now be getting telemetries from the target machine.
- To check, go to the **Apps** tab and select **Search & Reporting**.
![[splunk index 6.png]]

- We can skip the tour for this one.
![[splunk index 12.png]]
- In the search bar, type `index="endpoint"` make sure that the time range is set in the last 24 hours. Click the magnifying glass to begin searching.
![[services 9.png]]

- We now see events gathered from the target PC populating the search results.
![[splunk search 1.png]]

- Clicking on the **host** in the selected fields shows us what device or machine the data is being gathered from, and we see that it is from our target PC.
![[splunk search 2.png]]

### 3.2 On Windows Server
Configuring and installing Sysmon and Splunk Universal Forwarder on the Windows server is technically following the same steps that we did earlier on our target PC.