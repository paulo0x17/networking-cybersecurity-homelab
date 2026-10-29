
## What is a VLAN?

A VLAN (Virtual Local Area Network) is a way of dividing a physical connection virtually, isolating the network.

Since my network devices are connected to both my server and my personal computer, creating a VLAN and seperating those 2 will ensure better security and organization.

## Pre-requesites

Since I have 2 wired connections between my switch and my router, I will bundle them together into a Link Aggregation Group (LAG) using the open standard LACP (802.3ad). A LAG will be good for this setup because it bundles the two physical connections into one virtual connection, combining the speed (two 1Gbps ports become one 2Gbps logical pipe) and adding redundancy incase one of the connections fail.

First I will set it up on my router using the following commands:

```
/interface bonding add name=bond1 mode=802.3ad slaves=ether2,ether3
```

![VLAN 1](..pictures%20git/VLAN%201.png)

Then on the switch I will configure the rest of the LAG using LACP:
```
interface range GigabitEthernet1/0/14 - 15
channel-protocol lacp
channel-group 1 mode active 
no shutdown
```

Finally, configuring the logical Port-Channel interface as a trunk
```
interface Port-channel 1
switchport mode trunk 
switchport nonegotiate
```

Now the LAG is up and running, to confirm it we can run the following commands:

On the router CLI, I can confirm with
``/interface bonding monitor bond1
![Pasted image 20261028123922](..pictures%20git/Pasted%20image%2020261028123922.png)
They're both active

Now on the switch CLI I can confirm with
``show etherchannel summary
![527](..pictures%20git/Pasted%20image%2020261028124142.png)
It says (SU) meaning its working on Layer2 and that its in use

## Setup

Now that the LAG is setup I will create 2 vlans, VLAN 10 for my personal PC and VLAN 20 for my server. I will need to it up using the switch and the router so it can act as a router-on-a-stick, thus enabling inter-vlan routing. I will need inter-vlan routing to access the Proxmox GUI on my personal computer.

First I will create the VLANs on my Router:

```
/interface vlan
add name=vlan10-server vlan-id=10 interface=bond1
add name=vlan20-personal vlan-id=20 interface=bond1
```

Assign Gateway IP Addresses:
```
/ip address 
add address=192.168.10.1/24 interface=vlan10-server comment="Server Gateway" 
add address=192.168.20.1/24 interface=vlan20-personal comment="PC Gateway"
```

Set Up DHCP for VLAN 20:
```
/ip pool 
add name=pool-vlan20 ranges=192.168.20.10-192.168.20.250 

/ip dhcp-server 
add address-pool=pool-vlan20 interface=vlan20-personal name=dhcp-vlan20 disabled=no 

/ip dhcp-server network 
add address=192.168.20.0/24 gateway=192.168.20.1 dns-server=8.8.8.8,1.1.1.1
```

Now on the switch

Define the VLAN Database:
```
configure terminal 
vlan 10 name Server 
vlan 20 name Personal
```

Assign Access Ports for End Devices:
(Server)
```
interface GigabitEthernet1/0/2 
switchport mode trunk
switchport trunk allowed vlan 10 
no shutdown
```

(Personal PC)
```
interface GigabitEthernet1/0/4 
switchport mode access 
switchport access vlan 20 
no shutdown
```

After running ``ipconfig /release`` and ``ipconfig /renew``, I regained internet access alongside the VLAN IP
![Pasted image 20261028132047](..pictures%20git/Pasted%20image%2020261028132047.png)

Since I also changed the server's IP address, I need to assign a new static address so that it can be accessed through the browser.
```
nano /etc/network/interfaces
```

It shows:
```
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet dhcp
	bridge-ports nic0
	bridge-stp off
	bridge-fd 0
```

We will change it to:
```
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet manual
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094

auto vmbr0.10
iface vmbr0.10 inet static
    address 192.168.10.100/24
    gateway 192.168.10.1
    comment "Proxmox Server VLAN 10"
```

Save it then ``reboot``

Now I can access Proxmox Web GUI over the address ``https://192.168.10.100:8006/


