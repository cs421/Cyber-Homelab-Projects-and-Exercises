This PCAP investigation activity is a retired network analysis challenge from **Blue Team Labs Online** (https://blueteamslabs.online), and completed with guidance from **MYDFIR** (https://www.youtube.com/watch?v=0DuJnPxKfBg)

![[btlo challenges.png]]
![[btlo retired.png]]
# Scenario
The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system. Can you investigate and determine if this activity is malicious or not? You have been provided a PCAP, investigate using any tools you wish.

## Capture File Properties
View capture file properties to take note of the time frame.
![[btlo portscan capture file properties.png]]

```ad-important
Always ask the client this question:
"The PCAP you provided me is within this [time frame], is that correct?"
```


## Protocol Hierarchy
![[btlo portscan protocol hierarcy.png]]

Look out for the following protocols:
- SMB
- DNS
- SSH
- HTTP (cleartext)
They can be indicators of a potential lateral move opportunity.

## Conversations
### IPv4
![[blto portscan conversation.png]]
Take note of the top 2 conversations and IP addresses:
- **10.251.96.4** & **10.251.96.5**
- **172.20.10.5** & **172.20.10.2**

### TCP
![[blto portscan tcp conversation.png]]
Src IP (**10.251.96.4**) using the same src port (**41675**) to different dest ports of dest ip (**10.251.96.5**) indicates **port scanning**.

```ad-note
Ports should be unique **per connection attempt**. If the IP uses the same port in different connection attempts, it usually indicates **port scanning** activity.
```
![[btlo portscan port 80 connect.png]]
![[btlo portscan port 80 22 scan.png]]
- Scrolling down, we can see that **10.251.96.4** had multiple connections with port **80** (http).
- Going back to the top, we see that **10.251.96.4**'s connection attempt at port **80** consisted of 184 bytes, rather than the usual 118 bytes shown by other connection attempts on the list. This might mean that dest IP's port 80 had responded back with a **SYN/ACK** packet. 
- The same is seen on dest IP's port 22. port 22 is **SSH**.

This indicates that dest IP's port 80 and port 22 is **open**.
![[dest ip port 80 open.png]]

- Scrolling further down, we see that dest IP **10.251.96.5** had responded to src IP **10.251.96.4** on port **4422**. 
- There is also an external IP that starts with **34.122.121.31** connecting with dest IP on port 80. This can be a C2 server or just something legitimate.
We can hypothesize that dest IP **10.251.96.5** is a web server of some sort, but currently we cannot be sure

We can also see the second conversation between **172.20.10.5** and **172.20.10.2**.

### UDP
![[btlo portscan udp conversations.png]]

Checking the UDP section, we see DHCP and DNS requests and a few NetBIOS requests.

## Scanning PCAP

![[btlo wireshark change time.png]]
This changes the time info from duration of connection to the date and time of the day.

We will select the first HTTP connection, which is on packet **14**, and follow its HTTP stream.
![[btlo portscan first http.png]]
![[btlo portscan http stream.png]]

According to the HTTP stream, we can see that the IP **172.20.10.2** is an Ubuntu server.
![[btlo portscan packet 38 post.png]]
![[btlo portscan packet 38 http stream.png]]

Scrolling further down, we see the first **POST** packet in the pcap. following its http stream, we see credentials that were used to log in.

*username = admin&passwordAdmin%401234*

```ad-note
In POST requests, special characters are usually encoded. Just search up 'URL decoder' and paste the character you wish to decode. In the case above, the special character is the '%40' part of the password. Running it to the URL decoder will give us the character '@'.
```

![[btlo portscan first red connection.png]]

Scrolling further down, we see the first red connection, which is also the start of the conversation between the src IP **10.251.96.4** and dest IP **10.251.96.5**
![[btlo portscan add port details.png]]

This adds the source and destination port details in the main display.
![[btlo portscan packet 133.png]]

On packet 133 and 134, There was a SYN/ACK connection from dest IP **10.251.96.4** on port 80, confirming that port 80 is open on the dest IP.

![[btlo portscan packet 150.png]]

In packet 150 and 151, we can also confirm that port 22 on dest IP **10.251.96.4** is open.
![[btlo portscan packet 2172.png]]
![[btlo portscan packet 2172 htp stream.png]]

- In packet 2172, we see that this is the first communication between **10.251.96.5** and **10.251.96.4** on port 80. 
- Following the http stream, we can see that dest IP **10.251.96.5** is an Ubuntu server.
- We will also take note of the **User-Agent**: *Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0*
![[btlo portscan packet 2190.png]]
![[btlo portscan packet 2190 http stream.png]]

- Packet 2190 provides us with the first POST request from **10.251.96.4** to **10.251.96.5**.
- Following 2190's http stream, we see the following credentials:
	- *username=%27&password=%27*
		- Running **%27** in the URL decoder will give us the ( ' ) symbol.

## gobuster
![[btlo portscan packet 2215.png]]
![[btlo portscan packet 2215 http stream.png]]

- Scrolling further down, we see another GET request in packet 2215.
- Following the http stream, we see that the **User-Agent** has changed to: *gobuster/3.01*.
	- **gobuster** is a brute-force tool that can crawl between directories
- Time of **gobuster**'s first action: **16:34:05**

We will search for response code **200** from IP **10.251.96.5** using the filter: `(http.response.code == 200) && (ip.src==10.251.96.5)`
![[btlo portscan 200 filter result.png]]

Looking at the filter result, the usual length of packets vary from 500-700. But packet 7725 has a length of 8494, which means that the server is delivering something big to its destination (**10.251.96.4**).
![[btlo portscan packet 7725.png]]

- Following its http stream and scrolling down, we see the directory in which the server has responded from. 
- The directory's name is */info.php*, and it is an html file with *phpinfo* as the title.
![[btlo portscan packet 7725 apache version.png]]
- The version of php used is also visible in the info, which means that the attacker can be able to look for vulnerabilities to exploit in this particular version.
![[btlo portscan gobuster end.png]]
**gobuster** ended its search at **16:34:06**.

## SQLmap
![[btlo portscan packet 13979.png]]
![[btlo portscan packet 13979 http stream.png]]

Further down the list, we see another POST request from **10.251.96.4** to **10.251.96.5** at packet 13979
- Following the http stream, the **User-Agent** has changed to: *sqlmap/1.4.7*.
	- **sqlmap** is a tool that performs automated SQL attacks.
- They also tried to login with the following credentials: *username=user&password=pass*.
Time **sqlmap** was first encountered is at **16:36:17**

![[btlo portscan packet 14060.png]]
![[btlo portscan packet 14060 http stream.png]]
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20sql%20attack%20decode.png)

In packet 14060, we see an unusual POST request from **10.251.96.4** to **10.251.96.5**. 
- Following the http stream, we see a string of what could only be an SQL attack.
- Pasting the string into the URL decoder will give us the script that the attacker is trying to to.
	- Their end goal is to execute a command that will read */etc/passwd* as provided by the command `EXEC xp_cmdshell('cat ../../../etc/passwd')`
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20sqlmap%20last%20time.png)

The last POST request made by **sqlmap** is on packet 15978, which ended at **16:37:28**.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20filter%20post%20request.png)

We will search for POST requests made by **10.251.96.4** by the filter syntax: `http.request.method == POST && ip.addr == 10.251.96.4`.
- The result displayed that the last POST method made is on packet 16102 and occured at **16:40:39**.
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20last%20post%20http%20stream.png)

Following the http stream, the POST request was made on the */upload.php* file, and refered to by *editprofile.php*. 
- The attackers must have clicked on *editprofile.php*, which they clicked on */upload.php* that was inside.
- The filename that was uploaded was **dbfunctions.php**.

On packet 16134, we see a GET function performed on **dbfunctions.php?cmd=id**
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20get%20dbfunction.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20get%20uid.png)

Following the http stream, we see that the command is successful in getting the UIDs and GIDs.

The attacker initiated the **id** command at **16:40:51**
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20packet%2016201.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20packet%2016201%20http%20stream.png)
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%20python%20command.png)

On packet 16201, there is another GET request on **dbfunctions**, followed by encoded commands.
- Following the http stream, we see a clearer view of the command.
- Running the command into the url decoder reveals a python command that imports a connection to **10.251.96.4** on port **4422**, and a call for a subproces `["/bin/sh","-i"]`
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%204422%20initial%20connect.png)

Packet 16203 shows us the initial connection to port 4422
![](https://github.com/cs421/Cyber-Homelab-Projects-and-Exercises/blob/main/PCAP%20Investigations/attachments/btlo%20portscan%204422%20stream.png)

Following the tcp stream, we see that the attacker has successfully created a web shell (**dbfunctions.php**) in which they now have hands-on keyboard access on the server.
- Looking at the stream, they were listing directories and attempting to remove the files that they uploaded, but failed to do so.

```ad-tip
"In real life scenarios, persistence is likely to occur, such as creating a cron job, modifying ssh keys, or spinning up a cryptominer." - MyDFIR
```

# Challenge Submission

![challenge submission](https://github.com/cs421/Create_Homelab_Project/assets/152476259/ed3f2e7d-34b2-4da5-aaa4-d42341a5c5df)

