When I first setup everything I connected my ISP ethernet cable directly to the router and then from the router to the switch and from the switch to my computer and my server


## Bridge mode

I had to call my ISP to enable Bridge mode on my ISP's router so I could unlock the full potential of the Mikrotik Router.

After Bridge mode was turned on my ISP router I had to do some configuration on RouterOS, I had to allow remote requests and set the DNS servers to 1.1.1.1 and 8.8.8.8 to avoid a DNS blockage

![423](/Pictures/Router%204.png)


## DHCP 

My router was not distributing IP addresses so I had to setup a DHCP server.

![Router 2](/Pictures/Router%202.png)![375](/Pictures/Router%201.png)

I created a network pool named "dhcp_pool2" using 192.168.88.0/24 with the gateway 192.168.88.1 since thats the IP address of the router. I excluded the first 10 addresses of the pool as those are reserved for future scalability.

For DNS Servers I decided to use Cloudflare's 1.1.1.1 and Google's 8.8.8.8

After setting up everything I set the DHCP server to the "bridge1" interface 

![384](/Pictures/Router%203.png)

After setting up everything and doing ipconfig /renew on the cmd, I had access to the internet.

I made sure to assign a static IP address to my server just so I could consistently access the proxmox web GUI.


## Speed

I noticed that my internet's speed was considerably slower, after doing some research I did the following

```
/ip firewall filter add chain=forward action=fasttrack-connection connection-state=established,related

/ip firewall filter add chain=forward action=accept connection-state=established,related
```

Now, instead of inspecting each packet one by one it only inspects the first packet of a connection and lets the rest pass through. This reduces cpu usage and lets the router hit higher speed.

