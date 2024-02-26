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

#### Output result to TXT file

`cat access.log | cut -d ' ' -f1 | sort | uniq -c | sort -nr > IPs.txt`

![[ips txt.png]]
Take note of the top 2 IPs, **172.21.0.1** and **119.241.22.121**.
What are they doing? What activity is associated with the IPs?

### List User Agents
![[user agent.png]]
The user-agent detail is in the 6th column of the log and is surround by quotation marks , so we'll use the **cut** function to select them and use ' **"** ' as the delimiter.

`cat access.log | cut -d '"' -f 6 | sort | uniq -c | sort -nr`

![[cut f6.png]]

There are some details that are not related to user agents, such as the [Thu Jan 14....] data. We'll need to run an additional **cut** function, then use the " [ " as the delimiter on the first field after the first cut function

`cat access.log | cut -d '"' -f 6 | cut -d "[" -f 1 | sort | uniq -c | sort -nr`

![[cut f6 f1.png]]

#### Output result to TXT file
We can now output the user agent results to a text file called **"useragents.txt"**.

`cat access.log | cut -d '"' -f 6 | cut -d "[" -f 1 | sort | uniq -c | sort -nr > useragents.txt`

![[useragents txt.png]]
The top two user agents are **Linux** using **Mozilla 5.0** and **iPhone** using **Mozilla 5.0**.

![[sus user agents.png]]
We need to take note of these 3 user agents to dig deeper into later.

### POST Requests
Web traffic that have **POST** requests are usually worth looking into because they send data to the server.

`grep POST access.log`
![[grep post.png]]
![[403.png]]There are **POST** requests that return the code **403**. **403** are forbidden errors, so we can exclude them in the search.

`grep POST access.log | grep -v '403'`
`-v` - tells grep to exclude a string or a pattern

![[grep 403.png]]

#### List Source IPs with POST Requests
We can list and output the source IPs that made **POST** requests.
`grep 'POST' access.log | grep -v '403' | cut -d ' ' -f 1 | sort | uniq -c | sort -nr`
![[source ip post.png]]Take note of the 2nd IP, **103.69.55.212**, as this could be an IP of interest.

#### Output result to TXT file
We can now output the source IP results to a file called **"POSTIPs.txt"**.
`grep 'POST' access.log | grep -v '403' | cut -d ' ' -f 1 | sort | uniq -c | sort -nr > POSTIPs.txt`

![[output post ip.png]]

### OSINT
Perform OSINT on the 2nd IP (**103.69.55.212**)

![[abuseipdb.png]]**AbuseIPDB results:**
ISP: Quewu Co. Ltd.
Domain Name: pni.tw
Location: Taipei, Taiwan

![[taiwan ip virustotal.png]]No results on **VirusTotal**.

### Check Logs
Use `grep` to check log activity associated with **103.69.55.212**.
`grep '103.69.55.212' access.log`

![[103 69 55 212 grep.png]]![[fr34k.png]]
Upon scrolling down, we see a POST request for a file called "**fr34k.php**", which is suspicious.

![[plugin.png]]Scrolling back to the beginning of the log, we see GET requests on a couple of plugins: "**contact-form-7**" and "**simple-file-list**". Recalling the scenario, it is hypothesized that a plugin had a vulnerability for remote code execution in which the attacker might have exploited.

We can search for possible vulnerabilities about these plugins.

#### Contact Form 7
![[google contact form.png]]
![[contact form linkedin 1.png]]![[contact form linkedin 2.png]]

A LinkedIn post (https://www.linkedin.com/pulse/contact-form-7-wordpress-plugin-vulnerability-isha-singh) listed vulnerability details on the Contact Form 7 plugin, including the impact and ways to protect against it.

The summary indicates that attackers can upload arbitrary files because the plugin fails to properly sanitize user-supplied input.

![[contact form wpscan.png]]

A recent update on https://wpscan.com/plugin/contact-form-7/ indicates that the arbitrary file upload vulnerability on Contact Form 7 has been fixed in version 5.8.4, with a CVSS of 6.6 (medium)

#### Simple File List
![[google simple file list.png]]
![[exploitdb simple file list.png]]ExploitDB has a code of how the arbitrary file upload for Simple File List is executed.

We also see that the exploit is a **pre-authenticated Remote Code Execution**

### Filter Plugins
We will now filter the log results to show only activities associated with **contact-form-7** and **simple-file-list**, and find any activities regarding file uploads.

#### Contact Form 7
`grep 'POST' access.log | grep -i 'contact'`
`-i` - ignores case sensitivity
![[grep contact.png]]
The result for **contact-form-7** did not return any file upload activities.

#### Simple File List
`grep 'POST' access.log | grep -i 'simple'`
![[grep simple.png]]The results for **simple-file-list** has returned file upload activities, with the filename of **fr34k.php**.

We also see another IP (**119.241.22.121**) in the result, using a "**python-requests**" user agent. This could be an automated event.

#### 119.241.22.121

`grep '119.241.22.121' access.log`
![[grep 119 241 22 121.png]]
![[ip crawling.png]]
The first event recorded for this IP was on **Jan. 14 2021** at **05:42:34 UTC**.

We see interactions with the **contact-form-7** and **simple-file-list** plugins, and seems to be a crawling activity, performing GET requests on different files, such as **account%2ephp**. It also seems to be running through an iPhone, but it could be spoofed.

![[hb token.png]]
Scrolling down, we see a POST request to **itsec-hb-token=adminlogin**, which occured on **Jan 14 2021 05:54:14 UTC**.

This could mean that that attackers have identified a token that grants them access.

![[wpscan.png]]Further down, we also see the IP using the user agent **WPScan v3.8.10**, which occured on **Jan 14 2021 06:01:41 UTC**.

![[fr43k rename.png]]On **Jan 14 2021, 06:26:53 UTC**, we see a POST request for **simple-file-list**, then immediately we see a GET request for **fr34k.png**.

This could be the exploit's process of renaming **.png** files to **.php**. 

##### OSINT
###### AbuseIPDB
![[abuse 119 241 22 121.png]]
ISP: BIGLOBE Inc.
Domain: biglobe.co.jp
Location: Nagano, Japan

###### VirusTotal
![[virustotal 119 241 22 121.png]]

AbuseIPDB and VirusTotal did not return any results of malicious activity for **119.241.22.121**. 

### Checking First Activity
We will check the earliest date and time the two plugins were installed or activated

#### Contact Form 7
`grep -i 'contact-form-7' access.log`
![[contact form activate.png]]The **contact-form-7** plugin was first activated on **Jan 12 2021, 15:57:07 UTC**.

`grep -i 'simple-file-list' access.log`
![[simple file activate.png]]The **simple-file-list** plugin was first activated on **Jan 12 2021, 15:56:41 UTC**.

## Recap
###### January 12, 2021
**15:56:41 UTC**
Plugin: **contact-form-7** was activated

**15:57:07 UTC**
Plugin: **simple-file-list** was activated

*Both plugins have vulnerabilities to RCE*
contact-form-7 - version 5.3.1 and below
simple-file-list - version 4.2.2 and below

###### January 14, 2021
**05:42:32 UTC**
First External IP activity
IP: **119.241.22.121 - Japan**
*Interacting with both plugins*
*Crawling file paths on 172.21.0.3*

**05:54:14 UTC**
Identified Token: **adminlogin**
Full URI: **wp-login.php?itsec-hb-token=adminlogin**

**06:01:41 UTC**
IP: **119.241.22.121 - Japan**
Used tool: **WPScan (WordPress Scanner)**

**06:08:31 UTC**
IP: **103.69.55.212 - Taiwan**
*Crawling plugins on 172.21.0.3*

**06:26:53 UTC**
IP: **119.241.22.121 - Japan**
- Exploited plugin: **simple-file-list**
- Uploaded file named: **fr34k.png**
- GET Request towards **fr43k.php**

###### Last Observed Activity
**January 14, 2021 06:26:53 UTC**
IP: **119.241.22.121 - Japan**

**January 14, 2021 06:30:11 UTC**
IP: **103.69.55.212 - Taiwan**

Total Duration: **47 minutes and 37 seconds**

## Challenge Submission
![[wordpress challenge.png]]