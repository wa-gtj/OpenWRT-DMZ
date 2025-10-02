# OpenWRT-DMZ
This How to will show you how to set up a DMZ on an OpenWRT router.

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
In LuCI, go to Network -> interfaces. (Interfaces are not hardware ports, but software "connection points", connecting the hardware ports to the router creating the network.) 

To configure the hardware ports, we jump to the devices tab. You see a Bridge device called br-lan. Click on configure. deselect the Bridge ports (hardware poorts) You want to set aside for your DMZ. In my case it is only "lan4". Click on save.

Add a new device by clicking "Add device configuration...", select "bridge device" as the type, name it "br-dmz" and choose the bridge ports you removed from br-lan before.

Now, on the Interfaces tab, Add a new Interface, name it "DMZ", select static address as protocoll and select your newly created "br-dmz" device. hit create Interface.  Next you need to provide a IPv4 address for the Port. This needs to be different from the address of the LAN. I chose 192.168.100.1. (The "1" at the end is usually used for the upstream end (router to the outside world) or the device managing the network. Both cases are true here.)

In the "Firewall Settings" tab, we define a new zone called "DMZ"

Click Save. The Network is successfully created.

## 2 Firewall zone config

## 3 DHCP Server Setup

Network --> Interfaces; Edit "DMZ".

In the "DHCP Server" tab, we setup the DHCP Server.
