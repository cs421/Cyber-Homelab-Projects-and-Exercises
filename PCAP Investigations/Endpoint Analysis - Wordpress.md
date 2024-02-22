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

### List IPs
List the IPs associated with the log, specifically source IPs, by using the **cut** function

`cat access.log | cut -d ' ' -f 1`
`-d ' '` - use space as a delimiter
`-f 1` - selects the first field/column of the log

![[cut access log.png]]
This lists the IPs found in the first field, but does not sort them of duplicates or unique characters.

#### De-duplication
`cat access.log | cut -d ' ' -f 1 | sort uniq -c | sort -nr`
`sort | uniq -c` - sorts the duplicate strings or data and does a count of the duplicates
`sort -nr` - shows the top hits in descending order
![[access log sort uniq.png]]
There are **1035** hits for the IP **172.21.0.1**. There are also non-IP data, such as "[Thu]", "sh:", and "[Tue"

#### Output result to Txt file

`cat access.log | cut -d ' ' -f1 | sort | uniq -c | sort -nr > IPs.txt`

![[ips txt.png]]
Take note of the top 2 IPs, **172.21.0.1** and **119.241.22.121**.
What are they doing? What activity is associated with the IPs?

### List User Agents
![[user agent.png]]
The user-agent detail is in the 6th column of the log and is surround by quotation marks , so we'll use the **cut** function to select them and use ' **"** ' as the delimiter.

`cat access.log | cut -d '"' -f 6 | sort uniq -c | sort -nr`

![[cut f6.png]]

There are some details that are not related to user agents, such as the [Thu Jan 14....] data. We'll need to run an additional **cut** function, then use the " [ " as the delimiter on the first field after the first cut function

``
