This PCAP investigation activity is a retired network analysis challenge from **Blue Team Labs Online** (https://blueteamslabs.online), and completed with guidance from **MYDFIR** (https://www.youtube.com/watch?v=ypnIqtu5RjY)

![[blue team wordpress.png]]
![[download wordpress.png]]
# Scenario
One of our WordPress sites has been compromised but we're currently unsure how. The primary hypothesis is that an installed plugin was vulnerable to a remote code execution vulnerability which gave an attacker access to the underlying operating system of the server

### Access Log
Unzip the file to reveal the **access.log** file
![[wordpress unzip.png]]

Examine the first 10 events of the log by `head access.log`
![[head access log.png]]
###### First Event Recorded
IP Address: **127.21.0.1**
Date: **Jan 12, 2021**
Time: **15:52:41 UTC**

Examine the last 10 events of the log by `tail access.log`
![[tail access log.png]]
###### Last Event Recorded
IP Address: **127.21.0.1**
Date: **Jan 14, 2021**
Time: **07:46:52 UTC**