
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



