# OpenWRT-DMZ
This How-To will show you how to set up a DMZ on an OpenWRT router.

OpenWRT is an open source router firmware, based on Linux, running on a variety of hardware, even wellknown proprietary hardware. But there are also opensource hardware options including the official "OpenWRT One". OpenWRT provides sophisticated router capabilities and, very important, firewall capabilities. In addition more software can be installed to provide ad-blocking and other functionality. OpenWRT can be managed using the ssh or the graphic user interface called LuCI, which is used in this guide. For more information, please visit https://www.openwrt.org.

Personally, I run OpenWRT on an "AVM Fritz!Box 4040". AVM is a rather widely used brand for homenetwork routers in Germany.

## Goals
A DMZ (Demilitarized Zone) is a subnetwork (physical or logical) that contains the hosts (server) that need to e exposed to an untrusted network (e.g the internet). 

Of course, you could put the server in your home network, but in case the server is compromised, an attacker could infiltrate the whole network from there. So this is not a good idea.

Instead, the host, that needs to be exposed, is put in a seperate subnetwork. This subnetwork is then exposed to the untrusted network and seperately to the LAN. Ideally, there are firewall rules in place which prevent all but the absolute minimum of traffic to adjacent networks.

The same can be achieved using other router/firewall devices, for example of the relatively famous "netgate"-brand, which I have tested as well.

## Hardware choice
I want to limit the clutter in my homenetwork. One reason is certainly the power consumption, but also the mere goal of simplicity. That way I needed a device which has at least  3 Ethernet ports (1 WAN, 1 DMZ, 1 LAN) and can also provide Wireless connectivity.
In Addition the device needs a minimum of RAM and CPU Power to manage all this.

The AVM Fritz!Box 4040 has a dedicated WAN port and four LAN Ports (all 1 Gb), which can be assigned to different networks - which is just what I want.
In addition, it is relatively cheap: the prices for used hardware start at around 30 €. 

## What I want to achieve
  1 - separate Subnetwork named DMZ on seperate Ethernet port
  
  2 - by default all traffic into and out of the DMZ is rejected. The same applies to Forwards to other networks, WAN or LAN
  
  3 - DMZ hosts are allowed to get (static) IP-addresses from the router. (I still have to look into IPv6 connectivity - especially Global unicast address GUA)
  
  4 - DMZ hosts are reachable by ssh for management purposes from the LAN only (can be limited further to access from certain IP addresses)
  
  5 - DMZ hosts are allowed to update based on https-protocoll (can be time-restricted)

  6 - DMZ hosts are reachable from the WAN to serve their content

  6 a) - The Ports must be forwarded to the corresponding hosts in the DMZ
    
  6 b) - Traffic to the corresponding ports needs to be allowed by the firewall
    
  7 - DMZ hosts should be reachable from the LAN es well

  7 a) - the Domain names should resolve to the local IP addresses, so that the traffic is not routed via the WAN
    
  7 b) - Traffic to the corresponding ports needs to be allowed by the firewall

## 1 separate Subnetwork named DMZ on seperate Ethernet port
In LuCI, go to Network -> interfaces -> Devices

Configure "br-lan". Deselect the Bridge ports (hardware ports) that you want to set aside for your DMZ. In my case it is only "lan4". Click on save.

Add device configuration...", 
- device type: "bridge device"
- device name: "br-dmz" 
- bridge ports: choose the ports you removed from br-lan before.

Interfaces tab -> Add a new Interface
- name: "DMZ"
- protocol: static address
- device: "br-dmz"

-> hit create Interface.
- IPv4 address: make one up in the pattern of 192.168.XXX.1 (XXX between 0 and 254) This needs to be different from the address of the LAN. I chose 192.168.100.1.
- IPv4 netmask: 255.255.255.0
- leave the rest as it is on this tab
- "Firewall Settings" tab: define new zone called "DMZ"

Click Save and "Save & Apply" at the bottom. The Network is successfully created.

## 2 Firewall zone config
Go to Network -> Firewall

The newly created Zone DMZ shows up at the bottom.
Click Edit.
- Input: reject
- Output: reject
- Intra zone forward: reject
- leave the rest as it is on this tab. Especially, do NOT add a network to "Allow forward to destination zones" or "Allow forward from source zones". We want to allow only the minimum traffic

Save & Apply. Now the DMZ is completely isolated and locked up. Hosts won't even get an IP-Address.

## 3 DHCP Server Setup

Network --> Interfaces; Edit "DMZ". In the "DHCP Server" tab, we setup the DHCP Server.

Advanced Settings: Dynamic DHCP: uncheck (we want to assign only static addresses. -> Save

Network -> DHCP and DNS -> Devices & Ports:

Listen Interfaces: add checkmark before DMZ -> Save

switch Tab to "Static leases". click "Add".
- Hostname: (you can name it as you like, I prefer to use the subdomain I use for DynDNS, that should point to your Server, e.g. subdomain.domain.tld)
- MAC address: Enter the MAC-address of your Server
- IPv4 address: Enter an Address in the range of your DMZ-Interface. (change only the number after the last dot).

Save & Apply

The firewall still prohibits the Hosts on the DMZ network to aquire an IP address.

Go to Network -> Firewall -> Traffic Rules. Click Add at the bottom.
- Name: "DMZ - Allow DHCP"
- Protocol: "UDP"
- Source Zone: "DMZ"
- Destination Zone: "Device (input)"
- Destination Port: "67 547"
- Action: "accept"

-> Save and add another rule:
- Name: "DMZ - Allow DHCP"
- Protocol: "UDP"
- Source Zone: "Device (output)"
- Destination Zone: "DMZ"
- Destination Port: "68 548"
- Action: "accept"

-> Save and add another rule:
- Name: "DMZ - Allow ICMP"
- Protocol: "ICMP"
- Source Zone: "DMZ"
- Destination Zone: "Device (input)"
- Action: "accept"

-> Save and add another rule:
- Name: "DMZ - Allow ICMP"
- Protocol: "ICMP"
- Source Zone: "Device (output)"
- Destination Zone: "DMZ"
- Action: "accept"

Save & Apply.

We have created successfully four rules, so that Hosts on the DMZ Network get their IP from the router. disconnect the Ethernet-cable, wait ten seconds and reconnect it. It should work now. also basic funktions like ping should work. But any other traffic (like ssh or browsing is not working yet.

## 4 Allow SSH for Management/Maintainance
Add another rule at Network -> Firewall -> Traffic Rules. 
- Name: "DMZ - Allow SSH from LAN only"
- Protocol: "tcp"
- Source Zone: "lan"
- (Source address: if you want, you can specify the IP adresses that are allowed to connect to the hosts on DMZ)
- Destination Zone: "DMZ"
- Destination Port: "22" (or the port you configured the ssh-server to listen on)
- Action: "accept"

Save & Apply

## 5 Allow updates based on HTTPS
Debian updates via HTTPS. Make sure, that on your server, the /etc/apt/sources.list and files under /etc/apt/sources.list.d are pointing to https and not http.

Add another rule at Network -> Firewall -> Traffic Rules. 
- Name: "DMZ - Allow HTTPS"
- Protocol: "tcp"
- Source Zone: "DMZ"
- Destination Zone: "wan"
- Destination Port: "443 8443"
- Action: "accept"
- (On the Tab "Time Restrictions", you can restrict the ability to a certain time - useful if you schedule updates via cron.)

-> Save and add another rule (ignore, if you have already a DNS-Server on your Server running - add the specific rules instead):
- Name: "DMZ - Allow DNS-lookups"
- Protocol: "tcp"
- Source Zone: "DMZ"
- Destination Zone: "Device (input)"
- Destination Port: "53 853"
- Action: "accept"

Save & Apply


