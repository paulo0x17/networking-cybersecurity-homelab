
## What is NTP?

NTP is used to sync time across the network devices, its specially useful to get accurate reading on logs.

## Setup

We will create an NTP server on the router and make the switch be its client.

On the router we will enable NTP Client on MikroTik (to get accurate time from the internet):
```
/system ntp client set enabled=yes mode=unicast
/system ntp client servers add address=pool.ntp.org
```

Enable NTP Server on MikroTik (so the switch can query the router):
```
/system/ntp/server/set enabled=yes
```

On Winbox, System > NTP Server
![Pasted image 20261028201647](..pictures%20git/Pasted%20image%2020261028201647.png)
Confirming that ``Enabled`` is checked.


On the switch we will synchronize the NTP server:
```
configure terminal
ntp server 192.168.88.1 
clock timezone AZOT -1 0 
clock summer-time AZOST recurring last Sun Mar 1:00 last Sun Oct 2:00
```
