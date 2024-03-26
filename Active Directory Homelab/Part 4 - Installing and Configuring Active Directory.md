## Objective
- Install & configure **Active Directory** on the Windows Server, and promote the server as **Domain Controller**.
- Join the target Windows machine in the domain.

## 1. Install and Configure Active Directory
In the Windows server, we will first check its connectivity with the Internet and the Splunk server.

- Open up command prompt and `ping google.com` and `ping 192.168.10.10`

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ping%20google.png)

- In the Server Manager dashboard, click on **Manage** and select **Add Roles and Features**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%201.png)

- Click Next

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%202.png)

- In the **Installation Type**, select **Role-based or feature-based installation**, then click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%203.png)

- The **Server Selection** window will list your available servers if you have multiple. Right now the only server we have is the one we created. Select this and click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%204.png)

- In the **Server Roles** window, select **Active Directory Domain Services**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%205.png)

- Click **Add Features**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%206.png)

- The **Active Directory Domain Services** option is now checked. Click Next

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%207.png)

- Click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%208.png)

- Click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%209.png)

- Click on Install, and Active Directory will now begin installing.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%2010.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%2011.png)

- The installation will have successfully installed when you see the **Installation succeeded** message. We can now close the installation window.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/server%20manager%2012.png)

## 2. Promoting the server into a Domain Controller

- In the server manager dashboard, click on the flag with the yellow notification, and click on **Promote this server to a domain controller**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%201.png)

- In the **Deployment Configuration**, click the **Add a new forest** option, then create a name for the root domain. Click Next after.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%202.png)

```ad-note
The root domain name must be a **top-level domain**, which means you have to create a suffix for it (*.local*, *.test*, etc.).
```

- In the **Domain Controller Options**, create a password and leave everything else on default. Click Next after.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%203.png)

- Click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%204.png)

- Click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%205.png)

- This is the file path wherein the database file named *NTDS.dit* is saved. the file contains everything related to Active Directory, including password hashes. Click Next.

```ad-warning
If you see any unauthorized activity involving the *NTDS.dit*, your database and Active Directory is likely already compromised.
```

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%206.png)

- Click Next.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%207.png)

- Once the prerequisites have been validated, we can now click **Install**. The installation will begin and will reboot the system after installation.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%208.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%209.png)

- Click OK on this window for the system to restart and finish the installation.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%2010.png)

- After restarting, we can now see that our server has been promoted as a Domain Controller and the Active Directory has been installed.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/dc%2011.png)

## 3. Adding Users and Computers

- In the server manager dashboard, Click on **Tools** and select **Active Directory Users and Computers**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%201.png)

- Expanding down the domain, we see a directory called **Builtin**, and on the right side are the users that the server has automatically created.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%202.png)

- Double click on the user **Administrators** to see its properties.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%203.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%204.png)

- The **Members** tab lists the users who are members of the **Administrators** group.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%205.png)

- The **Members Of** tab lists the group/s that the **Administrators** group is a part of.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%206.png)

```ad-note
Builtin groups cannot be added to other groups, but a custom group can be added as a member of a builtin group.
```

- In the **Users** directory, we can see the list of users present in the server, along with their decriptions or roles.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%207.png)

In a real-world environment, users are delegated into different departments (IT, finance, HR, etc.). In Active Directory, these departments can be referred to as **Organizational Units**. 

We will be creating users and adding them to their respected organizational units.

- Right click on the domain, select **New**, and click **Organizational Unit**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%208.png)

- For the first one, we will name it *IT*.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%209.png)

- Right click on the *IT* directory, and click **New** -> **User**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2010.png)

- Name the user however you like and create a logon name for that user. Click Next after.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2011.png)

- Create a password for the user.
- In the real world, the **User must change password at next logon** must be **checked**, but for the sake of this homelab, we will uncheck this option.

- Click Next after.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2012.png)

- Click on **Finish** to create the user.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2013.png)

- Our first user has been created.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2014.png)

We can create a second Organizational Unit called *Finance* and another user within it.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2015.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2016.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%2017.png)

## 4. Joining the Target PC into the domain

- In the target PC, search for **This PC** and click **Properties**.
- Click **Advanced System Settings**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%201.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%202.png)

- Navigate to the **Computer Name** tab and click **Change** to change our domain.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%203.png)

- Select the **Domain** option and type the domain name. Click OK after.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%204.png)

- We will get this error stating that our AD domain cannot be contacted. This is because the target machine does not know how to resolve the AD domain.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%205.png)

We will need to change the target PC's DNS server to point out to the AD server.

- In the taskbar, click on the Network icon and click **Network & Internet settings**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%206.png)

- Click on **Change adapter options**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%207.png)


- Right click on the Ethernet and select **Properties**.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%208.png)

- Double click on **Internet Protocol Version 4 (TCP/IPv4)** to view the IP properties.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%209.png)

- On the DNS server address options, we see that it is currently pointing to Google's DNS server. 

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%2010.png)

- We need to change the value to our Active Directory's IP address, which is **192.168.10.7**. Click OK after.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%2011.png)

- To double check, open up command prompt and type `ipconfig /all` to list all IP configurations. The DNS server should now be pointing to the Active Directory's IP address.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%2012.png)

- Back in the **Computer Name** settings, Click OK again for the domain change. There should be no problem after this. 

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%204.png)

- We now need to log in as the AD admin to authorize the target PC's domain join.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%2013.png)

- The target PC has now successfully join the domain.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/join%20domain%2014.png)

- We are now prompted to restart the computer to apply changes. Close any running applications and restart the system.

- In the login page, we can now sign in as our newly created user within the Active Directory Domain.

![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%20login.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/Active%20Directory%20Homelab/attachments/ad%20user%20login%202.png)

