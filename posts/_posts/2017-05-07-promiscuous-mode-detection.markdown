---
layout: post
title: "Is the network device in promiscuous mode?"
date: 2017-05-07 21:25
comments: true
published: true
categories: networks devices
---

Wikipedia defines [**promiscuous mode**](https://en.wikipedia.org/wiki/Promiscuous_mode) as a mode for a wired network interface controller (NIC) or wireless network interface controller (WNIC) that causes the controller to pass all traffic it receives to the central processing unit (CPU)rather than passing only the frames that the controller is intended to receive. 

## Problem

Figuring out if a given network device is in promiscuous mode can be trickier than you'd think. 

At first glance, you'd think `iproute2` or `netstat -i` command should tell you if the device is in promiscuous mode but that's not always the case. 

We'll consider two examples here, first to show the case where it works as expected and second to show where it doesn't. Then we'll look into the reason behind this inconsistent behavior.

**A word on `netstat`:**

In netstat command, flag **`P`** is used to display if the interface is in promiscuous mode. However, **`P`** is also used for point to point connection. You can verify from the net-tools code [here](https://github.com/ecki/net-tools/blob/2617bbe4499749b93317cb41b2104278295eba81/lib/interface.c#L627-L634)

## Example 1: When it works
Following example, sets the promiscuous mode **on** using the `iproute2` and `netstat -i` command. You can verify it using the `iproute2` command and kernel logs.

{:.green_bold_text}
Verify that promiscuous mode is not enabled.
{% highlight sh %}
vagrant@precise64:~$ sudo ip link show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 
          qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:88:0c:a6 brd ff:ff:ff:ff:ff:ff

vagrant@precise64:~$ sudo netstat -i eth0
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR  TX-OK TX-ERR TX-DRP TX-OVR  Flg
eth0    1500 0    28880  0      0      0      17050    0      0      0   BMRU
{% endhighlight %}

{:.green_bold_text}
Enable the promiscuous mode.
{% highlight sh %}
vagrant@precise64:~$ sudo ip link set eth0 promisc on
{% endhighlight %}

{:.green_bold_text}
Check if promiscuous mode is enabled (see `PROMISC`) using `iproute2`
{% highlight sh %}
vagrant@precise64:~$ sudo ip link show eth0
2: eth0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 \
          qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:88:0c:a6 brd ff:ff:ff:ff:ff:ff

vagrant@precise64:~$ sudo netstat -i eth0
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR  TX-OK TX-ERR TX-DRP TX-OVR  Flg
eth0    1500 0    28880  0      0      0      17050    0      0      0   BMPRU

{% endhighlight %}

Let's check the kernel log messages, as logged in [__dev_set_promiscuity](http://elixir.free-electrons.com/linux/v3.10.105/source/net/core/dev.c#L4531) whenever a device is added/removed to/from promiscuous mode.

{% highlight sh %}
vagrant@precise64:~$ grep -r 'promiscuous' /var/log/kern.log
precise64 kernel: [44441.470885] device eth0 entered promiscuous mode
{% endhighlight %}



## Example 2: When it doesn't work

Consider this for example, adding an interface to bridge; set the promiscuous mode on for that interface. Check out the post [Linux Bridge - how it works](http://goyalankit.com/blog/linux-bridge) to learn more about bridge.

Following example creates a **bridge**, a **veth** pair and adds one end of the veth pair to bridge. According to [`br_add_if`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_if.c#L355), promiscuous mode is turned on for the interface.

{:.green_bold_text}
Create a new interface and add it to a bridge
{% highlight sh %}
vagrant@precise64:~$ sudo brctl addbr br0
vagrant@precise64:~$ sudo ip link add veth0 type veth peer name vpeer0
vagrant@precise64:~$ sudo brctl addif br0 veth0
{% endhighlight %}

{:.green_bold_text}
Check if promiscuous mode is enabled using `iproute2`
{% highlight sh %}
vagrant@precise64:~$ sudo ip -d link show veth0
9: veth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master \
 br0 state DOWN qlen 1000
    link/ether 12:30:e3:6b:42:2d brd ff:ff:ff:ff:ff:ff
    veth

vagrant@precise64:~$ sudo netstat -ian veth0
Kernel Interface table
Iface  MTU Met RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
veth0   1500 0   0      0      0 0             0      0      0      0 BM
vpeer0  1500 0   0      0      0 0             0      0      0      0 BM
{% endhighlight %}

It doesn't show the device to be in promiscuous mode as **`PROMISC`** is not set and the flag **P** is not present in **netstat**

Let's check kernel logs and see if the device was actually put into promiscuous mode.

{% highlight sh %}
vagrant@precise64:~$ grep -r 'promiscuous' /var/log/kern.log
precise64 kernel: [43656.288050] device veth0 entered promiscuous mode
{% endhighlight %}

As expected, **the device was in fact moved to promiscuous mode but `iproute2` doesn't show it in promiscuous mode.**

# Explanation
As you can see in netstat code [here](https://github.com/ecki/net-tools/blob/2617bbe4499749b93317cb41b2104278295eba81/lib/interface.c#L627-L628), it's checking to see if the `IFF_PROMISC` flag is set in interface flags. And to get the interface flags, netstat uses **`ioctl`** command **`SIOCGIFFLAGS`** in [`if_fetch`](https://github.com/ecki/net-tools/blob/2617bbe4499749b93317cb41b2104278295eba81/lib/interface.c#L419-L420).



# References

1. http://stackoverflow.com/questions/41678219/how-to-properly-put-network-interface-into-promiscuous-mode-on-linux?noredirect=1&lq=1
2. https://lists.gt.net/linux/kernel/178148
3. http://man7.org/linux/man-pages/man7/packet.7.html