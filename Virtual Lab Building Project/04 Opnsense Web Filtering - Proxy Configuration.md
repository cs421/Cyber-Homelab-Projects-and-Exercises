### Creating a trusted Certificate Authority

![[opnsense system trust.png]]

![[opnsense webca default.png]]
![[webca country.png]]

![[webca cert finish.png]]

### Installing the trusted CA

![[install cert.png]]

![[cert store local machine.png]]

![[cert browse location.png]]

![[cert trusted root.png]]
![[cert install finished.png]]


### Enabling Web Proxy

![[enable proxy.png]]

#### Transparent Proxy Configuration

![[transparent proxy firewall rule.png]]
![[transparent proxy firewall rule 02.png]]
![[port forward rule.png]]

![[opnsense forward proxy.png]]

![[authentication settings.png]]
![[blank authentication method.png]]
![[remote acl add.png]]
![[edit blacklist.png]]

**Blacklist URL:** ftp://ftp.ut-capitole.fr/pub/reseau/cache/squidguard_contrib/blacklists.tar.gz

```
For more info on Blacklists:
https://dsi.ut-capitole.fr/blacklists/index_en.php
```

![[download acl.png]]

![[blacklist edit acl.png]]
![[blacklist categories.png]]
![[blacklist deselect category.png]]

### Firewall setup

