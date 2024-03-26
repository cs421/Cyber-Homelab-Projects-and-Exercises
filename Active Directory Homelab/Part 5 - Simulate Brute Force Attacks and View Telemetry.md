## Objective
- Simulate a brute force attack on target PC via Kali Linux
- Setup and install **Atomic Red Team (ART)**
	- Run Atomic Tests
- View Telemetry via Splunk

## 1. Snapshots
Before proceeding with the attacks, make sure to take a snapshot of all your machines. If anything goes wrong with one or more machine/s, you will have your baseline configurations to revert back to.

- In your virtual machine, click on **Machine** and select **Take Snapshot**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/snapshot%201.png)

- You can rename the snapshot and add descriptions. Click OK when finished.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/snapshot%202.png)

- If you want to revert the machine to a last known good configuration via snapshot, click the X button on the machine and select **Restore current snapshot**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/snapshot%203.png)

## 2. Configuring Kali Linux

#### 2.1 Setting up Static IP
Per our diagram, the IP address on the attacker machine, aka Kali Linux, is **192.168.10.250**.

Log on to the Kali Linux virtual machine. The default credentials for both username and password is `kali`. 

- Right click on the ethernet icon and select **Edit Connections**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%201.png)

- Select **Wired connection 1** and click on the gear icon at the bottom left.


![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%202.png)

- Navigate to the **IPv4 Settings** tab, change the **Method** to "Manual", then click **Add**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%203.png)

- Type **192.168.10.250** for the address, **24** for the netmask, and **192.168.10.1** for the gateway.
- Type **8.8.8.8** for the DNS server, and click Save.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%204.png)

- In the desktop, right click and select **Open Terminal Here**

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%205.png)

- Type `ip a` to view the IP configurations.
- We see that our static IP hasn't been applied yet.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%206.png)

- To apply changes, click on the ethernet icon and select **Disconnect**

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%207.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%208.png)

- Then click on **Wired connection 1** to reconnect again.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%209.png)

- If we run `ip a` again, we now see that our IP address has been updated to **192.168.10.250**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%2010.png)

- We can check our connectivity by pinging Google and our Splunk server.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20ip%2011.png)

#### 2.2 Update dependencies

- We will now update the repositories and dependencies on our Kali machine by `sudo apt-get update && sudo apt-get upgrade -y`

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20update.png)

## 3. Brute Force Attack via Linux

We will now set up a directory which will contain the tools and files necessary for the brute force attack

- In the desktop, create a new directory by typing `mkdir ad-project` in the terminal.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20mkdir.png)

- We will now install **Crowbar**, a brute force tool used in penetration testing.

```ad-important
**DO NOT** target any assets that you do not have the permission to do so. Only use the tools for educational purposes and only target machines/assets within your own lab. 
```

- In the terminal, type `sudo apt-get install -y crowbar` to begin installation.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/kali%20install%20crowbar.png)

- After Crowbar has been installed, we will be using a wordlist called *rockyou.txt* for the brute forcing attack. *rockyou.txt* contains a list of popular  and commonly-used passwords 

- *rockyou.txt* is located at `/usr/share/wordlists/`. We can `cd` into it and view the directory.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/cd%20wordlist.png)

- We can unzip *rockyou* via gunzip by `sudo gunzip rockyou.txt.gz`

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rockyou%20unzip.png)

- We can now copy *rockyou.txt* into the **ad-project** directory back on our desktop by `cp rockyou.txt ~/Desktop/ad-project`.
- Go back to desktop and into the **ad-project** directory.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rockyou%20cp.png)

- We can view the file size of *rockyou.txt* by `ls -lh`

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rockyou%20size.png)

We see that *rockyou.txt* is 134MB in size, which means that it contains a lot of passwords, and using them all for brute-forcing will take a lot of time. Instead we will only use the first 20 passwords in the list.

- Using `head -n 20` will only output the first 20 passwords.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/head%20rockyou.png)

- We can append this output into a new file called *passwords.txt* by typing `head -n 20 rockyou.txt > passwords.txt`
- The `>` symbol outputs the previous result into a new file.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rockyou%20passwords.png)

- We can now see *passwords.txt* when we type `ls`, alongside *rockyou.txt*. Viewing the contents via `cat` shows that the first 20 passwords are present in *passwords.txt*.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/cat%20passwords.png)

In a real-world brute forcing attempt, an attacker may need to do some basic Active Directory attacks for reconnaissance, but in our scenario, we already know a certain password that we can target and use against the target PC. That password is what the user of the target PC has. We can add this password into the *passwords.txt* for our brute-forcing exercise.

- Type `nano passwords.txt` to edit the file.

- We will add the user's password at the very bottom of the list.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/add%20password.png)

- We can now save and exit the file by `ctrl+x` and `y`, then hit Enter.

- If we `cat` out *passwords.txt* again, we can now see that our password has been added to the list.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/add%20password%203.png)


## 4. Enabling Remote Desktop Protocol (RDP) on Target Machine

- On the Windows target PC, search for "This PC" and click on **Properties**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%201.png)

- Scroll down and click on **Advanced system settings**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%202.png)

- Log in with your adminstrator account.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%203.png)

- Navigate to the **Remote** tab and click on **Allow remote connections to this computer**. Then click on **Select Users**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%204.png)

- Click **Add**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%205.png)

- Type in the two usernames that we created. You can click **Check Names** to autofill the details.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%206.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%207.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%208.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%209.png)

- Click OK after to close the windows. Then click Apply and exit the **Properties** window.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/rdp%2010.png)


## 5. Using Crowbar

In our Kali machine, we'll open up a terminal and navigate to the *ad-project* directory.
- `crowbar -h` brings up the manual for Crowbar, as well as its description and usage.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/crowbar%201.png)

- To target the user in the target PC, we'll run crowbar with the following parameters: `crowbar -b rdp -u [username] -C passwords.txt -s [target source ip]/32`
	- `-b` - indicates the target service we want to use.
	- `-u` - the target username
	- `-C` - indicates that we want to use a wordlist.
	- `-s` - target server's IP.
		- /32 is a CIDR notation in which only 1 IP address is targeted throughout the attack.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/crowbar%202.png)

- Crowbar has started its attack on user **ssmith** within the target PC and has successfully logged on as *ssmith* with the password we inputted in *passwords.txt*.

### Splunk Brute Force Logs
In our Splunk web interface, log in and click on **Search & Reporting**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%201.png)

- In the search bar, we can type `index="endpoint" ssmith` to view events in the target PC involving the user *ssmith*. We can also change the time range to just the last 15 minutes. Click the magnifying glass to begin search.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%202.png)

- The results returned with 65 events involving *ssmith*. There is also a value within the **Interesting Fields** on the right called **EventCode**. Clicking this will open a pop up window that shows different Windows event IDs.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%203.png)

The two event IDs that we are interested in is **4625** and **4624**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%204.png)

```ad-note
The number of total events and the **4624** event ID counts that your run will produce may differ from mine because in my run, I had to troubleshoot a few mistakes when logging in and when I tried to run Crowbar, but essentially the key results will still be the same. 
```

- Searching for the **4625** event ID in https://ultimatewindowssecurity.com/securitylog/encyclopedia/ will show us that this event ID indicates that an account has failed to log on.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%205.png)

- In our Splunk results, the number of EventCode **4625** results is 20. This means that an account has failed to log on 20 times. Remember that we used *passwords.txt* as the password list to brute force with, and it has 20 passwords originally, excluding the real password that we added at the last minute.

- Clicking on **4625** will add the value to the search and will run again to produce the 20 events related to that event ID.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%206.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%207.png)

- Also, looking at the time the events happened, we see that the events almost happened at the same time after each other. This is an indication of a brute force attack.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%208.png)

- If we search for the event ID **4624**, it tells us that this indicates that an account was successfully logged on.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%209.png)

- Typing the event ID in our Splunk search will yield results for successful log ins.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%2010.png)

```ad-note
Please refer to my notes above regarding the number of events. If you ran Crowbar successfully on the first try, there should be only 1 result for EventCode **4624**.
```

- Scrolling to my results, there is a successful login attempt in the same time window that the brute force attack happened.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%2011.png)

- We can click on **Show all 70 lines** to view more details regarding the event.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%2012.png)

- If we scroll down to the **Network Information** field, we can see that the log in was done by our Kali machine, along with its IP of **192.168.10.250**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/splunk%20brute%2013.png)

## 6. Installing Atomic Red Team
**Atomic Red Team** is a library of MITRE ATT&CK framework tests used via PowerShell for penetration testing.

In our Windows target PC, we'll have to bypass the execution policy first to run scripts and set drive exclusions before installing **Atomic Red Team**.

#### 6.1 Bypass Execution Policy
- Open up PowerShell with administrator privileges.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%201.png)

- Type in `Set-ExecutionPolicy Bypass CurrentUser` and press "Y".

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%202.png)

#### 6.2 Add C: to Exclusions
- Next, we'll need to set exclusions to the *C:* Drive so we can install **ART**. In the taskbar, click on the up arrow and select **Windows Security**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%203.png)

- Click on **Virus & threat protection**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%204.png)

- Scroll down to **Virus & threat protection settings** and click **Manage settings**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%205.png)

- Scroll down to **Exclusions** and click **Add or remove exclusions**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%206.png)

- Log in with your administrator account, then click **Add an exclusion**, and select "Folder".

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%207.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%208.png)

- Click on *Local Disk (C:)* and click **Select Folder**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%209.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/exclusion%2010.png)

#### 6.3 Install Atomic Red Team
- Back in PowerShell, type the following after another to get **Atomic Red Team** from the repository and install it.

```powershell
IEX (IWR https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1 -UseBasicParsing);

Install-AtomicRedTeam -getAtomics
```

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/install%20art%201.png)

- Press "Y" and ENTER when you get this message:

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/install%20art%202.png)

- **Atomic Red Team** should be installed successfully.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/install%20art%203.png)

## 7. Running Atomic Red Team Tests

- The **ART** library is located at *C:\AtomicRedTeam\atomics*.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%201.png)

If we go to https://attack.mitre.org, we can see that these are the technique ID present in the framework.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%202.png)

- Back in the **ART** library, we can try and test the technique ID of **T1136**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%203.png)

- In the MITRE framework, this points out to the **Create Account** tactic under **Persistence**

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%204.png)

- Click beside the technique will also reveal the sub-techniques, and these techniques are also present in our **ART** library.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%205.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%206.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%207.png)

- We can try and test out the **T1136.001** technique.
- In PowerShell, type `Invoke-AtomicTest T1136.001` and hit ENTER to begin the test.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%208.png)

- Once the test has finished executing, take a look at the new user created by the test, *NewLocalUser*.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2011.png)

- We can look this up in our Splunk search.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%209.png)

- There are now events related to *NewLocalUser*.

We'll do another test involving commands executed in PowerShell.

- In the MITRE framework, under **Execution**, we can use the PowerShell subtechnique of the **Command and Scripting Interpreter** technique. Its technique ID is **T1059.001**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2010.png)

- We'll execute the test by typing `Invoke-AtomicTest T1059.001`

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2013.png)

- During this process, this will trigger the Windows Security threat protection.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2012.png)

- Once the process has finished, we'll take a note of this command made by the **T1059.001** test: `-exec bypass -noprofile "$Xml'`.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2014.png)

- Back in Splunk, we can search for any events related to PowerShell searching for `index="endpoint" powershell`.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2015.png)

- We may scroll down at the events for a bit, but in one event we will see the command `-exec bypass -noprofile"$Xml'` created by **ART**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20test%2016.png)

By recognizing certain behaviors or signatures of attacks, we can create alerts from this in the future. 

## 8. Troubleshooting Atomic Red Test

When you try to use Atomic Red Test in a new session, sometimes this error will occur:

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20import%20module.png)

For every new session, you will have to import the following module in order to use the `Invoke-AtomicTest` function.

```powershell
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/art%20import%20module%202.png)
