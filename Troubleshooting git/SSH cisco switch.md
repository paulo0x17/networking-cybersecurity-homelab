
Removed the Proxmox enterprise repository lists:
```
rm -f /etc/apt/sources.list.d/pve-enterprise.list
rm -f /etc/apt/sources.list.d/ceph.list
```

Added the free Proxmox community repository:
```
echo "deb http://download.proxmox.com/debian/pve trixie pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```

Commented out lingering enterprise repository references:
```
sed -i '/enterprise.proxmox.com/s/^/#/' /etc/apt/sources.list /etc/apt/sources.list.d/*.list 2>/dev/null
```

Successfully updated package lists and installed `screen`:
```
apt update && apt install screen -y
```

Find the usb connection from the switch to the server:
```
dmesg | grep -E "ttyUSB|ttyACM"
```

Launch screen with the name of the usb connection (in this case, ttyACM0):
```
screen /dev/ttyACM0 9600
```


After this, the switch's CLI appears. Now we will setup the switch to accept SSH connections.

First, I set a domain name alongside a hostname:
```
enable
configure terminal
hostname Switch
ip domain-name switch.local
```

Then I configure a management IP on VLAN 1:
```
interface vlan 1
ip address 192.168.88.2 255.255.255.0
no shutdown
```

Add the default gateway:
```
exit
ip default-gateway 192.168.88.1
```

Generate RSA encryption keys and enable SSH v2:
```
crypto key generate rsa modulus 2048
ip ssh version 2
```

Create an admin user account:
```
username admin privilege 15 secret *my password*
```

Restrict remote management lines to SSH only using local accounts:
```
line vty 0 15
transport input ssh 
login local
```

Finally we save the changes:
```
end 
write memory
```

