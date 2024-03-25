## Objective
- Install & configure **Active Directory** on the Windows Server, and promote the server as **Domain Controller**.
- Join the target Windows machine in the domain.

## 1. Install and Configure Active Directory
In the Windows server, we will first check its connectivity with the Internet and the Splunk server.
- Open up command prompt and `ping google.com` and `ping 192.168.10.10`
![[ping google.png]]
- In the Server Manager dashboard, click on **Manage** and select **Add Roles and Features**.
![[server manager 1.png]]

- Click Next
![[server manager 2.png]]

- In the **Installation Type**, select **Role-based or feature-based installation**, then click Next.
![[server manager 3.png]]

- The **Server Selection** window will list your available servers if you have multiple. Right now the only server we have is the one we created. Select this and click Next.
![[server manager 4.png]]

- In the **Server Roles** window, select **Active Directory Domain Services**.
![[server manager 5.png]]

- Click **Add Features**.
![[server manager 6.png]]

- The **Active Directory Domain Services** option is now checked. Click Next
![[server manager 7.png]]

- Click Next.
![[server manager 8.png]]

- Click Next.
![[server manager 9.png]]

- Click on Install, and Active Directory will now begin installing.
![[server manager 10.png]]
![[server manager 11.png]]

- The installation will have successfully installed when you see the **Installation succeeded** message. We can now close the installation window.
![[server manager 12.png]]

## 2. Promoting the server into a Domain Controller

- In the server manager dashboard, click on the flag with the yellow notification, and click on **Promote this server to a domain controller**.
![[dc 1.png]]

- In the **Deployment Configuration**, click the **Add a new forest** option, then create a name for the root domain. Click Next after.
![[dc 2.png]]
```ad-note
The root domain name must be a **top-level domain**, which means you have to create a suffix for it (*.local*, *.test*, etc.).
```

- In the **Domain Controller Options**, create a password and leave everything else on default. Click Next after.
![[dc 3.png]]

- Click Next.
![[dc 4.png]]

- Click Next.
![[dc 5.png]]

- This is the file path wherein the database file named *NTDS.dit* is saved. the file contains everything related to Active Directory, including password hashes. Click Next.
```ad-warning
If you see any unauthorized activity involving the *NTDS.dit*, your database and Active Directory is likely already compromised.
```
![[dc 6.png]]

- Click Next.
![[dc 7.png]]

- Once the prerequisites have been validated, we can now click **Install**. The installation will begin and will reboot the system after installation.
![[dc 8.png]]
![[dc 9.png]]

- Click OK on this window for the system to restart and finish the installation.

![[dc 10.png]]

- After restarting, we can now see that our server has been promoted as a Domain Controller and the Active Directory has been installed.
![[dc 11.png]]

## 3. Adding Users and Computers

- In the server manager dashboard, Click on **Tools** and select **Active Directory Users and Computers**.
![[ad user 1.png]]

- Expanding down the domain, we see a directory called **Builtin**, and on the right side are the users that the server has automatically created.
![[ad user 2.png]]

- Double click on the user **Administrators** to see its properties.
![[ad user 3.png]]
![[ad user 4.png]]

- The **Members** tab lists the users who are members of the **Administrators** group.
![[ad user 5.png]]

- The **Members Of** tab lists the group/s that the **Administrators** group is a part of.
![[ad user 6.png]]
```ad-note
Builtin groups cannot be added to other groups, but a custom group can be added as a member of a builtin group.
```

- In the **Users** directory, we can see the list of users present in the server, along with their decriptions or roles.
![[ad user 7.png]]

In a real-world environment, users are delegated into different departments (IT, finance, HR, etc.). In Active Directory, these departments can be referred to as **Organizational Units**. 

We will be creating users and adding them to their respected organizational units.

- Right click on the domain, select **New**, and click **Organizational Unit**.
![[ad user 8.png]]

- For the first one, we will name it *IT*.
![[ad user 9.png]]

- Right click on the *IT* directory, and click **New** -> **User**.
![[ad user 10.png]]

- Name the user however you like and create a logon name for that user. Click Next after.
![[ad user 11.png]]

- Create a password for the user.
- In the real world, the **User must change password at next logon** must be **checked**, but for the sake of this homelab, we will uncheck this option.
- Click Next after.
![[ad user 12.png]]

- Click on **Finish** to create the user.
![[ad user 13.png]]

- Our first user has been created.
![[ad user 14.png]]

We can create a second Organizational Unit called *Finance* and another user within it.

![[ad user 15.png]]
![[ad user 16.png]]
![[ad user 17.png]]

## 4. Joining the Target PC into the domain

- In the target PC, search for **This PC** and click **Properties**.
- Click **Advanced System Settings**.

![[join domain 1.png]]![[join domain 2.png]]

- Navigate to the **Computer Name** tab and click **Change** to change our domain.
![[join domain 3.png]]

- Select the **Domain** option and type the domain name. Click OK after.
![[join domain 4.png]]

- We will get this error stating that our AD domain cannot be contacted. This is because the target machine does not know how to resolve the AD domain.

![[join domain 5.png]]

We will need to change the target PC's DNS server to point out to the AD server.

- In the taskbar, click on the Network icon and click **Network & Internet settings**.
![[join domain 6.png]]

- Click on **Change adapter options**.
![[join domain 7.png]]


- Right click on the Ethernet and select **Properties**.
![[join domain 8.png]]

- Double click on **Internet Protocol Version 4 (TCP/IPv4)** to view the IP properties.
![[join domain 9.png]]

- On the DNS server address options, we see that it is currently pointing to Google's DNS server. 
![[join domain 10.png]]

- We need to change the value to our Active Directory's IP address, which is **192.168.10.7**. Click OK after.

![[join domain 11.png]]

- To double check, open up command prompt and type `ipconfig /all` to list all IP configurations. The DNS server should now be pointing to the Active Directory's IP address.

![[join domain 12.png]]

- Back in the **Computer Name** settings, Click OK again for the domain change. There should be no problem after this. 

![[join domain 4.png]]

- We now need to log in as the AD admin to authorize the target PC's domain join.
![[join domain 13.png]]

- The target PC has now successfully join the domain.

![[join domain 14.png]]

- We are now prompted to restart the computer to apply changes. Close any running applications and restart the system.

- In the login page, we can now sign in as our newly created user within the Active Directory Domain.

![[ad user login.png]]![[ad user login 2.png]]

