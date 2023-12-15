

![[opnsense ids ips activate.png]]


![[suricata custom nmap.png]]

![[opnsense enable ssh.png]]


`sudo apt-get install filezilla`

![[customnmap rules.png]]
![[customnmap xml.png]]

![[filezilla quickconnect.png]]

**Host:** `sftp://<opnsense ip address>`
**Port:** 22

![[filezilla trust host.png]]

![[filezilla browse folder.png]]

**Local**: Suricata rules location
**Remote site:** `/usr/local/opnsense/scripts/suricata/metadata/rules`

![[custom nmap copied.png]]

![[http server.png]]
![[opnsense ids restart.png]]
![[customnmap enable.png]]
![[customnmap enabled.png]]
![[opnsense rule view.png]]

![[sched tab.png]]

![[alerts tab.png]]



## Nmap

![[nmap opnsense.png]]

`-sS` - starts the SYN scan
`-Pn` - tells Nmap to still continue even if server is not responding to a ping
`--top-ports 500` - scans the top 500 ports. The default is **1000**

![[nmap result.png]]

![[opnsense nmap alert.png]]