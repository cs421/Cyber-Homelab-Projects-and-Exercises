## Virtual Machine List
**Windows 10** - Target machine

**Kali Linux** - Attacker machine

**Windows Server 2022** - Active Directory

**Ubuntu 22.04** - Splunk


## 1. Installing VirtualBox and Windows 10 virtual machine

Installing VirtualBox and the Windows 10 VM is very linear, as shown in MyDFIR's tutorial: https://www.youtube.com/watch?v=YxpUx0czgx4 from **2:07 - 8:34**
## 2. Installing Kali Linux
- Go to https://kali.org, and click on "Download"
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20org.png)

- Choose the **Virtual Machines** option
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20vm.png)

- Choose the 32-bit or 64-bit VirtualBox option, according to your PC specs.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20prebuilt.png)

- Extract the downloaded zip file
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/extract%20kali.png)

- Double-click on the **.vbox** file. It will automatically be added to VirtualBox.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20vbox.png)

- In the VirtualBox dashboard, we can see the newly added kali VM and the default login credentials.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20default.png)

## 3. Installing Windows Server

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/google%20windows%20server.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%2064bit.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20form.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/windows%20server%20english.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/vbox%20new.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/vbox%20new%20server%202022.png)

- Check the "Skip Unattended Installation" button
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20hardware.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20hard%20disk.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/vbox%20server%20start.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20start.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20install%20now.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20standard%20eval%20desktop.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20agreement.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20custom%20install.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20custom%20install%202.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20installing.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20create%20account.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20ctrl%20alt%20del.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/winserver%20dashboard.png)

## 4. Installing Ubuntu 22.04 Server for Splunk
- Go to https://ubuntu.com/server and click the download button
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20server%20download.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20server%20download%202.png)

- In VirtualBox, click the **New** button and name the new machine as "Splunk". Choose the destination folder, and select the ISO file we downloaded earlier.
- Keep the "Skip Unattended Installation" checked.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20server%20vbox.png)

- For the **Base Memory**, bump the amount to **8 GB**, and select **2** for the **Processors**. We will need more memory and processing power for this because this machine will ingest data and will be used for searching.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20server%20vbox%202.png)

- For the **Hard Disk**, bump the amount to **100 GB**, and click "Finish".
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20server%20vbox%203.png)

- Click on "Start" to power up the machine.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20server%20start.png)

- Select "**Try or Install Ubuntu Server**" and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/try%20install%20ubuntu%201.png)

- The machine will now proceed with the installation.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing.png)

- From this point onwards, we will be selecting the defaults for the options in the installation.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%202.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%203.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%204.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%205.png)

- Leave the Proxy address field blank and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%206.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%207.png)

- Navigate with the down arrow key and select Done by pressing Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%208.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%209.png)

- Select "Continue" with the down arrow key and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2010.png)

- Type any name you want, then name the server "**splunk**", and create a username and password.
- Select "Done" with the down arrow key and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2011.png)

- Select "Continue"
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2012.png)

- Leave the "Install OpenSSH server" unchecked, select "Done" with the down arrow key and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2013.png)

- Leave these options unchecked, select "Done" with the down arrow key and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2014.png)

- When you see the "**Reboot Now**" option, it means that the server has finished installing.
- Select "Reboot Now" with the down button and press Enter.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2015.png)

- Just press Enter when you encounter this screen.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20installing%2016.png)

- When the server has finished booting up, press Enter input your login credentials we created earlier
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20login.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20login%202.png)

- After logging in, we will be updating the files and dependencies with `sudo apt-get update && apt-get upgrade -y`
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20upgrade.png)

- Just press Enter when you encounter this screen. After that, the setup is now complete.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20upgrade%202.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ubuntu%20upgrade%203.png)
