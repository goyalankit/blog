---
layout: post
title: "Anatomy of the MACVLAN driver"
date: 2017-05-13 11:15
comments: true
published: true
categories: networks drivers macvlan
---




## Experimental setup

If you want to verify things for yourself or just play with it, you can create a new module with macvlan driver [source code](http://elixir.free-electrons.com/linux/v3.10.105/source/drivers/net/macvlan.c) and use [printk](https://en.wikipedia.org/wiki/Printk) to print various things in the driver. For example, `prink` can be placed in init method of the module.


{% highlight c %}
static int __init macvlan_init_module(void)
{
    int err;
    printk("<1>Running custom macvlan driver.\n");
    register_netdevice_notifier(&macvlan_notifier_block);
    ...
}
{% endhighlight %}

and then you should see the output in `dmesg`,
{% highlight sh %}
vagrant@precise64:~$ dmesg
[12869.172380] Running custom macvlan driver.
{% endhighlight %}


# `Macvlan` Driver workflow

## Load time
During the driver [load time](http://elixir.free-electrons.com/linux/v3.10.105/source/drivers/net/macvlan.c#L977), it registers itself to netdevice notifier chain by calling `register_netdevice_notifier`. 

Notification chains facilitate passing of messages between different modules by providing a _publisher-subscriber_ functionality. Any operation on **any net device** is delivered to all the subscribers. 


The driver registers  [`macvlan_device_event`](http://elixir.free-electrons.com/linux/v3.10.105/source/drivers/net/macvlan.c#L931) as the callback and in the callback it checks if the device is a macvlan port. If not, then it returns immediately with `NOTIFY_DONE`. If it is, then it handles the event accordingly.

After registering itself for notification events, it registers with routing netlink using `rtnl_link_register` which is netlink in `NETLINK_ROUTE` netlink family. Netlink socket is an asynchronous queue based communication channel between kernel space and user space. This socket based interface can be used to interact with macvlan driver. It registers callbacks for the following operations:

{% highlight c %}
static struct rtnl_link_ops macvlan_link_ops = {
	.kind		= "macvlan",
	.setup		= macvlan_setup,
	.newlink	= macvlan_newlink,
	.dellink	= macvlan_dellink,
};
{% endhighlight %}

`kind` is checked during registeration and is used to refer oper

References:

# References

1. [Notification chains in Linux](http://codingfreak.blogspot.com/2012/01/notification-chains-in-linux-part-01.html)<br/>
1. 