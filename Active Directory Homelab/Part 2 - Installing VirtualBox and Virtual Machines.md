## Virtual Machine List
**Windows 10** - Target machine
**Kali Linux** - Attacker machine
**Windows Server 2022** - Active Directory
**Ubuntu 22.04** - Splunk

## 1. Installing VirtualBox and Windows 10 virtual machine

Installing VirtualBox and the Windows 10 VM is very linear, as shown in MyDFIR's tutorial: https://www.youtube.com/watch?v=YxpUx0czgx4 from **2:07 - 8:34**
## 2. Installing Kali Linux
- Go to https://kali.org, and click on "Download"
![[kali org.png]]
- Choose the **Virtual Machines** option
![[kali vm.png]]
- Choose the 32-bit or 64-bit VirtualBox option, according to your PC specs.
 ![[kali prebuilt.png]]
- Extract the downloaded zip file
![[extract kali.png]]
- Double-click on the **.vbox** file. It will automatically be added to VirtualBox.
![[kali vbox.png]]
- In the VirtualBox dashboard, we can see the newly added kali VM and the default login credentials.
![[kali default.png]]
## 3. Installing Windows Server

![[google windows server.png]]
![[server 64bit.png]]![[server form.png]]
![[windows server english.png]]![[vbox new.png]]
![[vbox new server 2022.png]]
- Check the "Skip Unattended Installation" button
![[server hardware.png]]
![[server hard disk.png]]
![[vbox server start.png]]![[winserver start.png]]
![[winserver install now.png]]![[winserver standard eval desktop.png]]
![[winserver agreement.png]]![[winserver custom install.png]]
![[winserver custom install 2.png]]![[winserver installing.png]]
![[winserver create account.png]]![[winserver ctrl alt del.png]]
![[winserver dashboard.png]]

## 4. Installing Ubuntu 22.04 Server for Splunk
- Go to https://ubuntu.com/server and click the download button
![[ubuntu server download.png]]![[ubuntu server download 2.png]]
- In VirtualBox, click the **New** button and name the new machine as "Splunk". Choose the destination folder, and select the ISO file we downloaded earlier.
- Keep the "Skip Unattended Installation" checked.
![[ubuntu server vbox.png]]
- For the **Base Memory**, bump the amount to **8 GB**, and select **2** for the **Processors**. We will need more memory and processing power for this because this machine will ingest data and will be used for searching.
![[ubuntu server vbox 2.png]]
- For the **Hard Disk**, bump the amount to **100 GB**, and click "Finish".
![[ubuntu server vbox 3.png]]
- Click on "Start" to power up the machine.
![[ubuntu server start.png]]
- Select "**Try or Install Ubuntu Server**" and press Enter.
![[try install ubuntu 1.png]]
- The machine will now proceed with the installation.
![[ubuntu installing.png]]
- From this point onwards, we will be selecting the defaults for the options in the installation.
![[ubuntu installing 2.png]]![[ubuntu installing 3.png]]
![[ubuntu installing 4.png]]
![[ubuntu installing 5.png]]
- Leave the Proxy address field blank and press Enter.
![[ubuntu installing 6.png]]
![[ubuntu installing 7.png]]
- Navigate with the down arrow key and select Done by pressing Enter.
![[ubuntu installing 8.png]]
![[ubuntu installing 9.png]]
- Select "Continue" with the down arrow key and press Enter.
![[ubuntu installing 10.png]]
- Type any name you want, then name the server "**splunk**", and create a username and password.
- Select "Done" with the down arrow key and press Enter.
![[ubuntu installing 11.png]]
- Select "Continue"
![[ubuntu installing 12.png]]
- Leave the "Install OpenSSH server" unchecked, select "Done" with the down arrow key and press Enter.
![[ubuntu installing 13.png]]
- Leave these options unchecked, select "Done" with the down arrow key and press Enter.
![[ubuntu installing 14.png]]
- When you see the "**Reboot Now**" option, it means that the server has finished installing.
- Select "Reboot Now" with the down button and press Enter.
![[ubuntu installing 15.png]]
- Just press Enter when you encounter this screen.
![[ubuntu installing 16.png]]
- When the server has finished booting up, press Enter input your login credentials we created earlier
![[ubuntu login.png]]
![[ubuntu login 2.png]]
- After logging in, we will be updating the files and dependencies with `sudo apt-get update && apt-get upgrade -y`
![[ubuntu upgrade.png]]
- Just press Enter when you encounter this screen. After that, the setup is now complete.
![[ubuntu upgrade 2.png]]
![[ubuntu upgrade 3.png]]
