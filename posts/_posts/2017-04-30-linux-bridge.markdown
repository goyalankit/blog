---
layout: post
title: "Linux Bridge - how it works"
date: 2017-05-07 21:07
comments: true
published: true
categories: bridge networks
---


Linux bridge is a layer 2 virtual device that on its own cannot receive or transmit anything unless you bind one or more real devices to it. [source](http://shop.oreilly.com/product/9780596002558.do).

As [Anatomy of a Linux bridge](https://wiki.aalto.fi/download/attachments/70789083/linux_bridging_final.pdf) puts it, bridge mainly consists of four major components:

- **Set of network ports (or interfaces)**: used to forward traffic between end switches to other hosts in the network.
- **A control plane**: used to run Spanning Tree Protocol (STP) that calculates minimum spanning tree, preventing loops from crashing the network.
- **A forwarding plane**: used to process incoming input frames from the ports, forward them to the network port by making a forwarding decision based on the MAC learning database.
- **MAC learning database**: used to keep track of the host locations in the LAN.

For each unicast mac address, bridge maintains a mac learning database to decide which ports to forward based on MAC addresses, and if it can't find an entry for a given mac address, it will broadcast the frame to all ports except the one where it received the frame from.

There are two main configuration subsystems to do bridges:
- **ioctl**: This interface is used to create/destroy bridges and add/remove interfaces to/from a bridge.
- **sysfs**: Management of bridge and port specific parameters.

## Creating a bridge

Bridge can be created using `ioctl` command `SIOCBRADDBR`; as can be seen by `brctl` utiligy provided by bridge-utils.

{% highlight sh %}
vagrant@precise64:~$ sudo strace brctl addbr br1
execve("/sbin/brctl", ["brctl", "addbr", "br1"], [/* 16 vars */]) = 0
...
ioctl(3, SIOCBRADDBR, 0x7fff2eae9966)   = 0
...
{% endhighlight %}

Note that there is no device at this point to handle ioctl device, so the ioctl command is handled by a stub method: [`br_ioctl_deviceless_stub`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_ioctl.c#L351), which in turn calls [`br_add_bridge`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_if.c#L235). This method calls `alloc_netdev`, which is a macro that eventually calls [`alloc_netdev_mqs`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/core/dev.c#L5660).

{% highlight c %}
br_ioctl_deviceless_stub
  |- br_add_bridge
      |- alloc_netdev
           |- alloc_netdev_mqs  // creates the network device
              |- br_dev_setup // sets br_dev_ioctl handler               
{% endhighlight %}
`alloc_netdev` also initializes the new netdevice using the [`br_dev_setup`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_device.c#L335). This also includes setting up the bridge specific [`ioctl handler`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_ioctl.c#L379). If you look at the handler code, it handles ioctl command to add/delete interfaces.

{% highlight c %}
int br_dev_ioctl(struct net_device *dev, struct ifreq *rq, int cmd) {
    ...
    switch(cmd) {
	case SIOCBRADDIF:
	case SIOCBRDELIF:
		return add_del_if(br, rq->ifr_ifindex, cmd == SIOCBRADDIF);
    ...
    }
    ..
}

{% endhighlight %}

## Adding an interface
As it can be seen in `br_dev_ioctl`, bridge can be created using `ioctl` command `SIOCBRADDIF`. To confirm:


{% highlight sh %}
vagrant@precise64:~$ sudo strace brctl addif br0 veth0
execve("/sbin/brctl", ["brctl", "addif", "br0", "veth0"], [/* 16 vars */]) = 0
...
# gets the index number of virtual ethernet device.
ioctl(4, SIOCGIFINDEX, {ifr_name="veth0", ifr_index=5}) = 0
close(4)
 # add the interface to bridge.
ioctl(3, SIOCBRADDIF, 0x7fff75bfe5f0)   = 0
...
{% endhighlight %}

**`br_add_if`**

The [`br_add_if`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_if.c#L325) method creates and sets up the new interface/port for the bridge by allocating a new `net_bridge_port` object. The object initialization is particularly interesting, as it sets the interface to receive all traffic, adds the network interface address for the new interface to the forwarding database as the local entry and attaches the interface as the slave to the bridge device.

{% highlight c %}
/* Truncated version */
int br_add_if(struct net_bridge *br, struct net_device *dev)
{
	struct net_bridge_port *p;
	/* Don't allow bridging non-ethernet like devices */
    ...
	/* No bridging of bridges */
    ...
	p = new_nbp(br, dev);
    ...
	call_netdevice_notifiers(NETDEV_JOIN, dev);
	err = dev_set_promiscuity(dev, 1);
	err = kobject_init_and_add(&p->kobj, &brport_ktype, &(dev->dev.kobj),
				   SYSFS_BRIDGE_PORT_ATTR);
    ...
	err = netdev_rx_handler_register(dev, br_handle_frame, p);
    /* Make entry in forwarding database*/
	if (br_fdb_insert(br, p, dev->dev_addr, 0))
		...
    ...
}
{% endhighlight %}

Some things worth noting in `br_add_if`:

- Only ethernet like devices can be added to bridge, as bridge is a layer 2 device.
- Bridges cannot be added to a bridge.
- New interface is set to promiscuous mode: `dev_set_promiscuity(dev, 1)`

The promiscuous mode can be confirmed from kernel logs.
{% highlight sh %}
vagrant@precise64:~$ grep -r 'promiscuous' /var/log/kern.log
precise64 kernel: [ 5185.751666] device veth0 entered promiscuous mode
{% endhighlight %}

Finally, `br_add_if` method calls `netdev_rx_handler_register`, that sets the `rx_handler` of the interface to [`br_handle_frame`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_input.c#L153)

After this method finishes, you have an interface (or port) in bridge.

---

## Frame Processing

{:.image_size_600}
![bridge frame processing](https://gist.githubusercontent.com/goyalankit/6ea0ea8448ad1946e0791b308970a5d3/raw/cb59f23f7017edd47c075817aa2cdabb76bfa79d/frame_processing_bridge2.png)
Frame processing starts with device-independent network code, in [`__netif_receive_skb`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/core/dev.c#L3579) which calls the `rx_handler` of the interface, that was set to [`br_handle_frame`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_input.c#L153) at the time of adding the interface to bridge.

The `br_handle_frame` does the initial processing and any address with prefix `01-80-C2-00-00` is a control plane address, that may need special processing. From the comments in `br_handle_frame`:

{% highlight sh %}
    /*
    * See IEEE 802.1D Table 7-10 Reserved addresses
    *
    * Assignment		 		Value
    * Bridge Group Address		01-80-C2-00-00-00
    * (MAC Control) 802.3		01-80-C2-00-00-01
    * (Link Aggregation) 802.3	        01-80-C2-00-00-02
    * 802.1X PAE address		01-80-C2-00-00-03
    *
    * 802.1AB LLDP 		01-80-C2-00-00-0E
    *
    * Others reserved for future standardization
    */
{% endhighlight %}

In the method, note that stp messages are either passed to upper layers or forwarded if STP is enabled on the bridge or disabled respectively. Finally if a forwarding decision is made, the packet is passed to `br_handle_frame_finish`, where the actual forwarding happens.

Here's the highly truncated version of [`br_handle_frame_finish`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_input.c#L60):

{% highlight c %}
/* note: already called with rcu_read_lock */
int br_handle_frame_finish(struct sk_buff *skb)
{
    struct net_bridge_port *p = br_port_get_rcu(skb->dev);
    ...
	/* insert into forwarding database after filtering to avoid spoofing */
	br = p->br;
	br_fdb_update(br, p, eth_hdr(skb)->h_source, vid);
	if (p->state == BR_STATE_LEARNING)
		goto drop;
	/* The packet skb2 goes to the local host (NULL to skip). */
	skb2 = NULL;
	if (br->dev->flags & IFF_PROMISC)
		skb2 = skb;
	dst = NULL;
	if (is_broadcast_ether_addr(dest))
		skb2 = skb;
	else if (is_multicast_ether_addr(dest)) {
        ...
	} else if ((dst = __br_fdb_get(br, dest, vid)) &&
			dst->is_local) {
		skb2 = skb;
		/* Do not forward the packet since it's local. */
		skb = NULL;
	}
	if (skb) {
		if (dst) {
			br_forward(dst->dst, skb, skb2);
		} else
			br_flood_forward(br, skb, skb2);
	}
	if (skb2)
		return br_pass_frame_up(skb2);
out:
	return 0;
    ...
}
{% endhighlight %}

As you can see in above snippet of `br_handle_frame_finish`, 
- An entry in forwarding database is updated for the source of the frame.
- (not in the above snippet) If the destination address is a multicast address, and if the multicast is disabled, the packet is dropped. Or else message is received using `br_multicast_rcv`
- Now if the promiscuous mode is on, packet will be delivered locally, irrespective of the destination.
- For a unicast address, we try to determine the port using the forwarding database (`__br_fdb_get`).
- If the destination is local, then `skb` is set to null i.e., packet will not be forwarded.
- If the destination is not local, then based on if we found an entry in forwarding database, either the frame is forwarded (`br_forward`) or flooded to all ports (`br_flood_forward`).
- Later, packet is delivered locally (`br_pass_frame_up`) if needed (based on either the current host being the destination or the net device being in promiscuous mode).

[`br_forward`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_forward.c#L120) method either clones and then deliver (if it is also to be delivered locally, by calling `deliver_clone`), or directly forwards the message to the intended destination interface by calling [`__br_forward`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_forward.c#L87).

[`bt_flood_forward`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_forward.c#L212) forwards the frame on each interface by iterating through the list in [`br_flood`](http://elixir.free-electrons.com/linux/v3.10.105/source/net/bridge/br_forward.c#L212) method.

# Conclusion

Bridges can be used to create various different network topologies and it's important to understand how they work. I have seen bridges being used with containers where they are used to provide networking in network namespaces along with `veth` devices. In fact the default networking in docker is provided using [bridge](https://docs.docker.com/engine/userguide/networking/#default-networks).

This is all for now, hopefully this was useful. This was mainly based on the excellent paper [Anatomy of a Linux bridge](https://wiki.aalto.fi/download/attachments/70789083/linux_bridging_final.pdf) and my own reading of linux kernel code. I'd appreciate any feedback, or comments.

---

Ah, the wonderful world of bridges.
{:.image_size_600}
![Bridge](https://gist.githubusercontent.com/goyalankit/6ea0ea8448ad1946e0791b308970a5d3/raw/bfa531da40571bdd72c21b752abb6768d346cb70/sf_bridge.jpg)


---

## References:
[Handwritten notes](http://goyalankit.com/blog/linux-bridge-notes)

[1] [Anatomy of a Linux bridge](https://wiki.aalto.fi/download/attachments/70789083/linux_bridging_final.pdf)<br/>
[2] [Understanding Linux Networking Internals - Christian Benvenuti](http://shop.oreilly.com/product/9780596002558.do)<br/>
[3] [Linux kernel v3.10.105 source code](http://elixir.free-electrons.com/linux/v3.10.105/source)
