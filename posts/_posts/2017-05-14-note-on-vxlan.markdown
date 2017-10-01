---
layout: post
title: "Why VXLAN (rfc7348)?"
date: 2017-06-24 10:04
comments: true
published: false
categories: networks vxlan
---

From the [rfc7348](https://tools.ietf.org/html/rfc7348#page-5) 
> VXLAN is a Layer 2 overlay scheme on a Layer 3 network.

To understand VXLAN better, let's first understand what's subnetting and VLAN.

Let's say we have a physical LAN where there are multiple hosts with IPs in `10.1.2.0/24` network. Each host can talk to the other host using a switch alone. Now, we want to group set of hosts and separate them from each other. What are our options?

# IP Subnetting
The first thought that comes to my mind is IP subnetting. It can prevent two hosts in different subnets from talking to each other unless we explicitly allow it using a router.

Say now we have two subnets, `10.1.2.0/25` where the two ranges are `10.1.2.1 - 10.1.2.126` and `10.1.2.129  - 10.1.2.254`. The broadcast address for first subnet is `10.1.2.127` and for second subnet it's `10.1.2.255`.

 However, **they still have the same layer 2 broadcast domain**. 
 
 Consider the following example (in gif below) where even though the host (`10.1.2.2`) can't ping the other hosts (`10.1.2.130` and `10.1.2.134`) but it can broadcast in the other subnet since the broadcast domain at layer 2 is same and switch will broadcast all the frames with destination `FFFF.FFFF.FFFF` to all the ports irrespective of layer 3 source/destination IPs.

{:.image_size_600}
![bradcast switch](https://gist.githubusercontent.com/goyalankit/df3686b62ac9bfd20f5eb292c02697bd/raw/3225ce3acb8e1d2b8a83da34f6028ac6fbaef9a7/broadcast_switch.gif)

The behavior is same in presence of router that allows the two subnets to talk to each other. Since switch operates at Layer 2.

{:.image_size_600}
![icmp own subnet broadcast](https://gist.githubusercontent.com/goyalankit/df3686b62ac9bfd20f5eb292c02697bd/raw/cc30fb145049c25966637c02e11d01f8277ff8d3/icmp_own_broadcast.gif)

# VLAN
So how can we split the broadcast domains for the subnets? This is where VLAN comes in. VLAN can be used to separate the broadcast domains as well. 

VLAN frames are tagged by switch based on what port they arrive at. VLAN header is 4 bytes long and is placed before the type field in ethernet frame. It contains a 12 bit VLAN Identifier (VID) that identifies the frame it belongs to.


In the example below, hosts (`10.1.2.3`, `10.1.2.2` and `10.1.2.135`) are in the same vlan (`vlan1`) and the remaining hosts belong to a different vlan. As in the previous experiments, let's ping the broadcast address (`10.1.2.127`) from the host `10.1.2.3`. Note that in this case, the packet is only broadcasted to the hosts that belong to the same vlan irrespective of the subnet. Note that the host (`10.1.2.35`) will drop the packet since the destination belongs to a different subnet and we don't have a default gateway set. However, hosts in the other vlan don't even receive the **icmp** request.

{:.image_size_600}
![vlan broadcast](https://gist.githubusercontent.com/goyalankit/df3686b62ac9bfd20f5eb292c02697bd/raw/30cf4ba8859f9dd0ab171cce7a43c41f779e71bc/vlan_broadcast.gif)

Note that 12 bit identifier limits the maximum number of vlans you can have to 4096. However with recent influx of virtualized environments, this is not enough and VLAN is not friendly enough for large number of multi tenant topologies. Hence the proposal for VXLAN.

# VXLAN

VXLAN stands for Virtual eXtensible Local Area Network, it's a layer 2 in layer 3 overlay tunnel. More specifically an Ethernet in UDP tunnel. The idea of VXLAN is similar to VLAN, as in it provides logical separation of private networks.

Consider the network topology in the diagram below network A and B are part of the same private network separated by external network. A-P, B-Q use same subnet in the same physical network.
![](https://gist.githubusercontent.com/goyalankit/df3686b62ac9bfd20f5eb292c02697bd/raw/72eaedd22c441ad96b5864555901f5f1fd6347bc/vxlan3.png)

The requirement from underlay network (UDP) is that it should have ip connectivity between two networks, UDP port [4789](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=4789) is reserved for VXLAN.

The endpoint of the tunnel, **VTEP (Virtual Tunnel End Point)** forms the control plane of VXLAN (overlay network). It maintains a mapping of internal MAC address to the outer IP address i.e., while sending a packet from A: 10.0.0.1 to B: 10.0.0.2. VTEP can learn 

VMs in subnets (A and B in the above picture) are unaware of the VXLAN themselves, and they route the traffic as they normally would. Say, a VM (A, 10.0.0.1) wants to route traffic to another VM in same network (B, 10.0.0.2) located at a different physical server and network. It would send the MAC frame as it would if both were on the same physical network. VTEP on that host checks the MAC address of 10.0.0.2

Following packet trace shows the VXLAN packet wrapper in UDP outer packet. UDP forms the overlay network and inner nodes (with IP 10.0.0.1) don't need to know about overlay network.

{:.image_size_600}
![](https://gist.githubusercontent.com/goyalankit/df3686b62ac9bfd20f5eb292c02697bd/raw/d1920b1f0cabfb35a5e350de61f1e4535d39cc7f/vxlan_packet_trace_1.png)

{:.image_size_600}
![](https://gist.githubusercontent.com/goyalankit/df3686b62ac9bfd20f5eb292c02697bd/raw/d1920b1f0cabfb35a5e350de61f1e4535d39cc7f/vxlan_packet_trace_2.png)

# Conclusion
VXLAN is an important piece of puzzle when it comes to scaling ipv4 and providing network abstractions. I hope you have a slightly better understanding of where VXLAN belongs in plethora of networking technologies.


## References:

[1] [Packet trace for VXLAN - cloudshark](https://www.cloudshark.org/captures/670aeb7bad79)<br/>
[2] [RFC-7348: Virtual eXtensible Local Area Network (VXLAN)](https://tools.ietf.org/html/rfc7348)

