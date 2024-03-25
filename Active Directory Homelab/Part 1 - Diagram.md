
![[Active Directory Diagram.jpg]]

- **Domain** - We can name our domain however we want, with our network IP of *192.168.10.0/24*.
- **Target machine** - Our target machine will be the Windows 10 machine, with the IP created by DHCP. It will have **Sysmon** installed for log collection and **Splunk Universal Forwarder** for forwarding logs to Splunk. We will also install **Atomic Red Team** in the target machine so we can generate telemetry from the MITRE framework.
- **Attacker machine** - Our attacker machine will be Kali Linux, with the IP of *192.168.10.250*.
- **Active Directory** - Our Active Directory server will be using Windows Server 2022 with the IP of *192.168.10.7*. It will also have **Sysmon** and **Splunk Universal Forwarder**.

