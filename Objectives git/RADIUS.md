## What is RADIUS?

RADIUS is a networking protocol and server solution designed to centralize Authentication, Authorization, and Accounting (AAA) for users and devices seeking to access network resources. 

It's responsible for:

- Centralized Authentication
- Authorization and dynamic VLAN assignment
- Session Accounting and tracking
- Integration with directory services (Active Directory, LDAP, etc.) 


On my homelab, running a RADIUS server allows me to implement enterprise-grade 802.1X network authentication for Wi-Fi and switches.

I will be using PacketFence, its both a [[repo/Objectives git/Network Access Control (NAC)]] and a RADIUS server.

In order to even setup the NAC I'll need to setup the RADIUS first, so lets start with that.

## OS Setup

I will deploy a Debian Virtual Machine in Proxmox
![[Pasted image 20261028212037.png]]

I will name it "PacketFence"
![[Pasted image 20261028212119.png]]

Then I will choose the Debian 12 ISO
![[Pasted image 20261028212946.png]]

Click Next

Then make sure Qemu Agent is ticked
![[Pasted image 20261028213139.png]]
It has many benefits, primarily lets the Proxmox VE host talk directly to the guest operating system.

Next

For this OS I will use 60gb of dedicated disk, like so:
![[Pasted image 20261028213842.png]]

Next

I allocate 4 cores:
![[Pasted image 20261028214238.png]]

Next

I allocate 8gb of RAM:
![[Pasted image 20261028214329.png]]

Next

I leave it as default:
![[Pasted image 20261028214358.png]]


Review and click Finish
![[Pasted image 20261028214757.png]]

On the left, I click 100 (PacketFence)
![[Pasted image 20261028214911.png]]

And hit Start to begin the OS installation
![[Pasted image 20261028214936.png]]


IMAGES



## Inside the Debian OS


