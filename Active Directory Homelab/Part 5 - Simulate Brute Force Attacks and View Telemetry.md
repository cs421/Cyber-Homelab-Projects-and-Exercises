## Objective
- Simulate a brute force attack on target PC via Kali Linux
- Setup and install **Atomic Red Team (ART)**
	- Run Atomic Tests
- View Telemetry via Splunk

## 1. Snapshots
Before proceeding with the attacks, make sure to take a snapshot of all your machines. If anything goes wrong with one or more machine/s, you will have your baseline configurations to revert back to.

- In your virtual machine, click on **Machine** and select **Take Snapshot**.
![[snapshot 1.png]]

- You can rename the snapshot and add descriptions. Click OK when finished.
![[snapshot 2.png]]

- If you want to revert the machine to a last known good configuration via snapshot, click the X button on the machine and select **Restore current snapshot**.
![[snapshot 3.png]]

## 2. Configuring Kali Linux

#### 2.1 Setting up Static IP
Per our diagram, the IP address on the attacker machine, aka Kali Linux, is **192.168.10.250**.

Log on to the Kali Linux virtual machine. The default credentials for both username and password is `kali`. 

- Right click on the ethernet icon and select **Edit Connections**.
![[kali ip 1.png]]

- Select **Wired connection 1** and click on the gear icon at the bottom left.
![[kali ip 2.png]]

- Navigate to the **IPv4 Settings** tab, change the **Method** to "Manual", then click **Add**.
![[kali ip 3.png]]

- Type **192.168.10.250** for the address, **24** for the netmask, and **192.168.10.1** for the gateway.
- Type **8.8.8.8** for the DNS server, and click Save.
![[kali ip 4.png]]

- In the desktop, right click and select **Open Terminal Here**
![[kali ip 5.png]]

- Type `ip a` to view the IP configurations.
- We see that our static IP hasn't been applied yet.
![[kali ip 6.png]]

- To apply changes, click on the ethernet icon and select **Disconnect**
![[kali ip 7.png]]
![[kali ip 8.png]]

- Then click on **Wired connection 1** to reconnect again.
![[kali ip 9.png]]

- If we run `ip a` again, we now see that our IP address has been updated to **192.168.10.250**.
![[kali ip 10.png]]

- We can check our connectivity by pinging Google and our Splunk server.
![[kali ip 11.png]]

#### 2.2 Update dependencies

- We will now update the repositories and dependencies on our Kali machine by `sudo apt-get update && sudo apt-get upgrade -y`
![[kali update.png]]

## 3. Brute Force Attack via Linux

We will now set up a directory which will contain the tools and files necessary for the brute force attack

- In the desktop, create a new directory by typing `mkdir ad-project` in the terminal.
![[kali mkdir.png]]

- We will now install **Crowbar**, a brute force tool used in penetration testing.

```ad-important
**DO NOT** target any assets that you do not have the permission to do so. Only use the tools for educational purposes and only target machines/assets within your own lab. 
```

- In the terminal, type `sudo apt-get install -y crowbar` to begin installation.
![[kali install crowbar.png]]

- After Crowbar has been installed, we will be using a wordlist called *rockyou.txt* for the brute forcing attack. *rockyou.txt* contains a list of popular  and commonly-used passwords 
- *rockyou.txt* is located at `/usr/share/wordlists/`. We can `cd` into it and view the directory.
![[cd wordlist.png]]

- We can unzip *rockyou* via gunzip by `sudo gunzip rockyou.txt.gz`
![[rockyou unzip.png]]

- We can now copy *rockyou.txt* into the **ad-project** directory back on our desktop by `cp rockyou.txt ~/Desktop/ad-project`.
- Go back to desktop and into the **ad-project** directory.
![[rockyou cp.png]]

- We can view the file size of *rockyou.txt* by `ls -lh`
![[rockyou size.png]]

We see that *rockyou.txt* is 134MB in size, which means that it contains a lot of passwords, and using them all for brute-forcing will take a lot of time. Instead we will only use the first 20 passwords in the list.

- Using `head -n 20` will only output the first 20 passwords.
![[head rockyou.png]]

- We can append this output into a new file called *passwords.txt* by typing `head -n 20 rockyou.txt > passwords.txt`
- The `>` symbol outputs the previous result into a new file.
![[rockyou passwords.png]]

- We can now see *passwords.txt* when we type `ls`, alongside *rockyou.txt*. Viewing the contents via `cat` shows that the first 20 passwords are present in *passwords.txt*.
![[cat passwords.png]]

In a real-world brute forcing attempt, an attacker may need to do some basic Active Directory attacks for reconnaissance, but in our scenario, we already know a certain password that we can target and use against the target PC. That password is what the user of the target PC has. We can add this password into the *passwords.txt* for our brute-forcing exercise.

- Type `nano passwords.txt` to edit the file.
- We will add the user's password at the very bottom of the list.
![[add password.png]]

- We can now save and exit the file by `ctrl+x` and `y`, then hit Enter.

- If we `cat` out *passwords.txt* again, we can now see that our password has been added to the list.
![[add password 3.png]]


## 4. Enabling Remote Desktop Protocol (RDP) on Target Machine

- On the Windows target PC, search for "This PC" and click on **Properties**.
![[rdp 1.png]]

- Scroll down and click on **Advanced system settings**.
![[rdp 2.png]]

- Log in with your adminstrator account.
![[rdp 3.png]]

- Navigate to the **Remote** tab and click on **Allow remote connections to this computer**. Then click on **Select Users**.
![[rdp 4.png]]
- Click **Add**.
![[rdp 5.png]]

- Type in the two usernames that we created. You can click **Check Names** to autofill the details.
![[rdp 6.png]]
![[rdp 7.png]]
![[rdp 8.png]]
![[rdp 9.png]]
- Click OK after to close the windows. Then click Apply and exit the **Properties** window.
![[rdp 10.png]]


## 5. Using Crowbar

splunk
search and reporting - index endpoint ssmith
eventcode - 4625 - 4624 - ultimatwindowssecurity


In our Kali machine, we'll open up a terminal and navigate to the *ad-project* directory.
- `crowbar -h` brings up the manual for Crowbar, as well as its description and usage.
![[crowbar 1.png]]

- To target the user in the target PC, we'll run crowbar with the following parameters: `crowbar -b rdp -u [username] -C passwords.txt -s [target source ip]/32`
	- `-b` - indicates the target service we want to use.
	- `-u` - the target username
	- `-C` - indicates that we want to use a wordlist.
	- `-s` - target server's IP.
		- /32 is a CIDR notation in which only 1 IP address is targeted throughout the attack.
![[crowbar 2.png]]
- Crowbar has started its attack on user **ssmith** within the target PC and has successfully logged on as *ssmith* with the password we inputted in *passwords.txt*.

### Splunk Brute Force Logs
In our Splunk web interface, log in and click on **Search & Reporting**.
![[splunk brute 1.png]]

- In the search bar, we can type `index="endpoint" ssmith` to view events in the target PC involving the user *ssmith*. We can also change the time range to just the last 15 minutes. Click the magnifying glass to begin search.
![[splunk brute 2.png]]
- The results returned with 65 events involving *ssmith*. There is also a value within the **Interesting Fields** on the right called **EventCode**. Clicking this will open a pop up window that shows different Windows event IDs.
![[splunk brute 3.png]]

The two event IDs that we are interested in is **4625** and **4624**.
![[splunk brute 4.png]]

```ad-note
The number of total events and the **4624** event ID counts that your run will produce may differ from mine because in my run, I had to troubleshoot a few mistakes when logging in and when I tried to run Crowbar, but essentially the key results will still be the same. 
```

- Searching for the **4625** event ID in https://ultimatewindowssecurity.com/securitylog/encyclopedia/ will show us that this event ID indicates that an account has failed to log on.
![[splunk brute 5.png]]
- In our Splunk results, the number of EventCode **4625** results is 20. This means that an account has failed to log on 20 times. Remember that we used *passwords.txt* as the password list to brute force with, and it has 20 passwords originally, excluding the real password that we added at the last minute.
- Clicking on **4625** will add the value to the search and will run again to produce the 20 events related to that event ID.
![[splunk brute 6.png]]
![[splunk brute 7.png]]

- Also, looking at the time the events happened, we see that the events almost happened at the same time after each other. This is an indication of a brute force attack.
![[splunk brute 8.png]]

- If we search for the event ID **4624**, it tells us that this indicates that an account was successfully logged on.
![[splunk brute 9.png]]

- Typing the event ID in our Splunk search will yield results for successful log ins.
![[splunk brute 10.png]]
```ad-note
Please refer to my notes above regarding the number of events. If you ran Crowbar successfully on the first try, there should be only 1 result for EventCode **4624**.
```

- Scrolling to my results, there is a successful login attempt in the same time window that the brute force attack happened.
![[splunk brute 11.png]]

- We can click on **Show all 70 lines** to view more details regarding the event.
![[splunk brute 12.png]]

- If we scroll down to the **Network Information** field, we can see that the log in was done by our Kali machine, along with its IP of **192.168.10.250**.
![[splunk brute 13.png]]

## 6. Installing Atomic Red Team
**Atomic Red Team** is a library of MITRE ATT&CK framework tests used via PowerShell for penetration testing.

In our Windows target PC, we'll have to bypass the execution policy first to run scripts and set drive exclusions before installing **Atomic Red Team**.


ps admin - set execution
winsecurity - manage settings - exlusion - c drive

c - atomicredteam

#### 6.1 Bypass Execution Policy
- Open up PowerShell with administrator privileges.
![[exclusion 1.png]]

- Type in `Set-ExecutionPolicy Bypass CurrentUser` and press "Y".
![[exclusion 2.png]]


#### 6.2 Add C: to Exclusions
- Next, we'll need to set exclusions to the *C:* Drive so we can install **ART**. In the taskbar, click on the up arrow and select **Windows Security**.
![[exclusion 3.png]]

- Click on **Virus & threat protection**.
![[exclusion 4.png]]


- Scroll down to **Virus & threat protection settings** and click **Manage settings**.
![[exclusion 5.png]]

- Scroll down to **Exclusions** and click **Add or remove exclusions**.
![[exclusion 6.png]]

- Log in with your administrator account, then click **Add an exclusion**, and select "Folder".
![[exclusion 7.png]]
![[exclusion 8.png]]

- Click on *Local Disk (C:)* and click **Select Folder**.
![[exclusion 9.png]]
![[exclusion 10.png]]

#### 6.3 Install Atomic Red Team
- Back in PowerShell, type the following after another to get **Atomic Red Team** from the repository and install it.
```powershell
IEX (IWR https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1 -UseBasicParsing);

Install-AtomicRedTeam -getAtomics
```

![[install art 1.png]]
- Press "Y" and ENTER when you get this message:
![[install art 2.png]]

- **Atomic Red Team** should be installed successfully.
![[install art 3.png]]

## 7. Running Atomic Red Team Tests

- The **ART** library is located at *C:\AtomicRedTeam\atomics*.
![[art test 1.png]]

If we go to https://attack.mitre.org, we can see that these are the technique ID present in the framework.

![[art test 2.png]]

- Back in the **ART** library, we can try and test the technique ID of **T1136**.
![[art test 3.png]]

- In the MITRE framework, this points out to the **Create Account** tactic under **Persistence**
![[art test 4.png]]

- Click beside the technique will also reveal the sub-techniques, and these techniques are also present in our **ART** library.
![[art test 5.png]]
![[art test 6.png]]
![[art test 7.png]]

- We can try and test out the **T1136.001** technique.
- In PowerShell, type `Invoke-AtomicTest T1136.001` and hit ENTER to begin the test.
![[art test 8.png]]

- Once the test has finished executing, take a look at the new user created by the test, *NewLocalUser*.
![[art test 11.png]]

- We can look this up in our Splunk search.
![[art test 9.png]]
- There are now events related to *NewLocalUser*.

We'll do another test involving commands executed in PowerShell.

- In the MITRE framework, under **Execution**, we can use the PowerShell subtechnique of the **Command and Scripting Interpreter** technique. Its technique ID is **T1059.001**.

![[art test 10.png]]

- We'll execute the test by typing `Invoke-AtomicTest T1059.001`
![[art test 13.png]]

- During this process, this will trigger the Windows Security threat protection.
![[art test 12.png]]

- Once the process has finished, we'll take a note of this command made by the **T1059.001** test: `-exec bypass -noprofile "$Xml'`.
![[art test 14.png]]

- Back in Splunk, we can search for any events related to PowerShell searching for `index="endpoint" powershell`.
![[art test 15.png]]

- We may scroll down at the events for a bit, but in one event we will see the command `-exec bypass -noprofile"$Xml'` created by **ART**.
![[art test 16.png]]

By recognizing certain behaviors or signatures of attacks, we can create alerts from this in the future. 


## 8. Troubleshooting Atomic Red Test

When you try to use Atomic Red Test in a new session, sometimes this error will occur:

![[art import module.png]]

For every new session, you will have to import the following module in order to use the `Invoke-AtomicTest` function.

```powershell
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```
![[art import module 2.png]]