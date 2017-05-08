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

At first glance, you'd think `iproute2` command should tell you if the device is in promiscuous mode but that's not always the case.

## Case 1: When it works
If you set the promiscuous mode **on** using the `iproute2` command, you can verify it using the `iproute2` command. For example,

{:.green_bold_text}
Check that promiscuous mode is not enabled.
{% highlight sh %}

vagrant@precise64:~$ sudo ip link show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 
          qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:88:0c:a6 brd ff:ff:ff:ff:ff:ff

{% endhighlight %}

{:.green_bold_text}
Enable the promiscuous mode.
{% highlight sh %}
vagrant@precise64:~$ sudo ip link set eth0 promisc on
{% endhighlight %}

{:.green_bold_text}
Check that promiscuous mode is enabled (`PROMISC` is set). 
{% highlight sh %}
vagrant@precise64:~$ sudo ip link show eth0
2: eth0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 \
          qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:88:0c:a6 brd ff:ff:ff:ff:ff:ff
{% endhighlight %}

Let's check the kernel log messages, as logged in [__dev_set_promiscuity](http://elixir.free-electrons.com/linux/v3.10.105/source/net/core/dev.c#L4531) whenever a device is added/removed to/from promiscuous mode.

{% highlight sh %}
vagrant@precise64:~$ grep -r 'promiscuous' /var/log/kern.log
precise64 kernel: [44441.470885] device eth0 entered promiscuous mode
{% endhighlight %}



## Case 2: When it doesn't work

Consider this for example, adding an interface to bridge; set the promiscuous mode on for that interface. Check out the post [Linux Bridge - how it works](http://goyalankit.com/blog/linux-bridge) to learn more about bridge.

Following example creates a **bridge**, a **veth** pair and adds one end of the veth pair to bridge. According to [`br_add_if`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_if.c#L355), promiscuous mode is turned on for the interface.

{% highlight sh %}
vagrant@precise64:~$ sudo brctl addbr br0
vagrant@precise64:~$ sudo ip link add veth0 type veth peer name vpeer0
vagrant@precise64:~$ sudo brctl addif br0 veth0
{% endhighlight %}

let's see what `iproute2` says about that:
{% highlight sh %}
vagrant@precise64:~$ sudo ip -d link show veth0
9: veth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master \
 br0 state DOWN qlen 1000
    link/ether 12:30:e3:6b:42:2d brd ff:ff:ff:ff:ff:ff
    veth
{% endhighlight %}

It doesn't show the device to be in promiscuous mode.

Let's check kernel logs and see if the device was actually put into promiscuous mode.

{% highlight sh %}
vagrant@precise64:~$ grep -r 'promis' /var/log/kern.log
precise64 kernel: [43656.288050] device veth0 entered promiscuous mode
{% endhighlight %}

As expected, **the device was in fact moved to promiscuous mode but `iproute2` doesn't show it in promiscuous mode.**


# Explaination

