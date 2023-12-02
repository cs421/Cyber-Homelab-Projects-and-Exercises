
```ad-note
Pfsense Download Link:
https://www.pfsense.org/download/
```

#### Pfsense Download Settings

![pfsense download settings](https://github.com/cs421/Create_Homelab_Project/assets/152476259/7c19a014-00c9-48e9-a686-38fcbfcc6783)

#### Install either VMware Workstation Pro or Oracle Virtual Box

In this instance, since I couldn't afford to purchase the pro version of VMware workstation, I used Virtual Box.

#### Setting up Pfsense in Virtual Box

For the ISO image, browse for the downloaded **pfsense iso** file

![[virtualbox_pfsense_setup_01.png]]

Set base memory to **2GB** and click **next**

![[virtualbox_pfsense_setup_02.png]]

Select **Do Not Add a Virtual Hard Disk** and click **Next**

![[virtualbox_pfsense_setup_03.png]]

Click **Finish**

![[virtualbox_pfsense_finish.png]]

Click **Continue**

![[virtualbox_pfsense_setup_final.png]]

#### Adding Network Adapters

In VirtualBox, select the **Pfsense** machine, click on **Settings**, then click on **Network**.

![[pfsense network adapters.png]]

We will be adding six network adapters, but VirtualBox only offers four adapters as the default amount.

To add more than four network adapters, we will need to use the CLI to enter commands.

To use the CLI, open command prompt as admin, then navigate to the VirtualBox directory, usually in **C:\Program Files\Oracle\VirtualBox**

The code to add a network adapter is `vmboxmanage modifyvm <machine_name> --nic[N]= <network>`

`<machine_name>` is the name of the virtual machine. In this case, it's **Pfsense**.

`[N]` is the adapter number you want to configure or to add. In our case, we need to add two more network adapters, so we would replace `[N]` with **5** and **6**.

`<network>` is the network type used by each network adapter. We will use **nat** for the type being, as we will configure this later.

![[pfsense nic5.png]]
![[pfsense nic6.png]]

We can now see the newly added adapters back in the dashboard.

![[pfsense added adapters.png]]

We will now configure the network types of each network adapter. We'll leave **Adapter 1** to **NAT** as default.
Click on the network type on **Adapter 2** to change it. Select **Internal Network** and rename the network. For continuity's sake, we will name each internal network as *vmnet2*, *vmnet3*, and so on.
![[vmnet.png]]

#### Booting Up Pfsense

Click **Start** to boot up **Pfsense** in VirtualBox

![[pfsense boot.png]]


![[pfsense default.png]]
![[pfsense install_01.png]]
![[psense install_02.png]]
![[pfsense install_03.png]]
![[pfsense install_04.png]]
![[pfsense install_05.png]]
![[pfsense_install 06.png]]
![[pfsense install_07.png]]
![[Pasted image 20231202184726.png]]