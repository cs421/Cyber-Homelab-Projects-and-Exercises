This lab consists of my documentation in creating a homelab network running on an Active Directory server, with multiple users, a Windows 10 PC as a target machine, and a Kali Linux machine as the attacker, and Splunk as the SIEM tool.

## Tools to be used:
- **VirtualBox** (https://virtualbox.org) - virtual machine
	- **Windows Server 2022** as the server with Active Directory
	- **Kali Linux** - attacker machine
	- **Windows 10 Pro** - target machine
- **Splunk** (www.splunk.com) - SIEM tool
	- **Splunk Universal Forwarder** - log forwarder
- **Sysmon** - log collection for Windows
- **Atomic Red Team** (https://atomicredteam.io/) - a library of tests mapped to the MITRE ATT&CK framework. 

I will be creating this homelab following along with MYDFIR's YouTube videos: https://www.youtube.com/watch?v=5OessbOgyEo

