---
layout: post
title: "Netfilter connection tracking - conntrack"
date: 2017-05-13 09:50
comments: true
published: false
categories: ipc kernel modules
---

Conntrack itself never alters packets.
Uses netfilter hooks to look at packets as they come in/leave
userspace can subscribe to ct events.

`iptables -m conntrack` doesn't do any connection tracking.  It only checks the packet state as determined by the eearlier connection tracking step.


conntrack states:
ESTABLISHED - packet matches existing entry.
NEW - first packet of a connection. But not placed in table until all iptables rules are passed.
RELATED - same as new, except packet relates to 
UNTRACKED - packet was intentionally not tracked.
INVALID - packet not seen or rejected by I4 trackers.


Main connection tracking table
- Lockless for read accesses.
- Can do parallel access.
- fix size, sysctl: net.netfilter.nf_conntrack_buckets, net.netfilter.nf_conntrack_max
- Each entry is hashed twice (original + reply) to deal with nat.

Conntrack extensions
- outside main nf_conn struct
- saves memory for less used features.

Net is built over connection tracking engine.
- NAT mappings are set up at conntrack creation time.
- ... why is iptables 'nat' table only sees first packet of flow.??
- one extra hash table: nat bysource table.
- used to ensure addr:port is unique when adding new mapping.
- all connections have nat mapping once a nat hook is active.

Overflow handling
nf_conntrack: table full, dropping packet
- main assumption: most enteries are not assured. assured: flag set by l4 protocol tracker at certain point (tcp: 3 way handshake is completed.)

we can only ever drop not assured entry.
doesn't play nice with nat. 
> long timeouts by default.
> increase resources.

what can we do about it? Remove some strange conntrack error handling:
If packet is INVALID, we always pass it onto the iptables, users can decide what to do.
In case we can't allocate, we always NF_DROP. (users can't change it)
Doesn't solve table exhaustion problem for all cases e.g., non-tracked packets. Very large conntrack timeouts (5 days).


------------------


Netfilter framework:
packet filtering and manipulation framework. Access to packet through 5 hooks. 

Return value must be one of these 5 methods.
```
/* Responses from hook functions. */
#define NF_DROP 0
#define NF_ACCEPT 1
#define NF_STOLEN 2
#define NF_QUEUE 3
#define NF_REPEAT 4
#define NF_STOP 5	/* Deprecated, for userspace nf_queue compatibility. */
#define NF_MAX_VERDICT NF_STOP
```

[Return codes](https://elixir.free-electrons.com/linux/v4.14.2/source/include/uapi/linux/netfilter.h#L10)


----

nf_conntrack_tuple
This is used to represent uni-directional flow.
```
/* The manipulable part of the tuple. */
struct nf_conntrack_man {
	union nf_inet_addr u3;
	union nf_conntrack_man_proto u;
	/* Layer 3 protocol */
	u_int16_t l3num;
};

/* This contains the information to distinguish a connection. */
struct nf_conntrack_tuple {
	struct nf_conntrack_man src;

	/* These are the parts of the tuple which are fixed. */
	struct {
		union nf_inet_addr u3;
		union {
			/* Add other protocols here. */
			__be16 all;

			struct {
				__be16 port;
			} tcp;
			struct {
				__be16 port;
			} udp;
			struct {
				u_int8_t type, code;
			} icmp;
			struct {
				__be16 port;
			} dccp;
			struct {
				__be16 port;
			} sctp;
			struct {
				__be16 key;
			} gre;
		} u;

		/* The protocol. */
		u_int8_t protonum;

		/* The direction (for tuplehash) */
		u_int8_t dir;
	} dst;
};


```
https://elixir.free-electrons.com/linux/v4.14.2/source/include/net/netfilter/nf_conntrack_tuple.h#L37
