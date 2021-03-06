---
layout: post
title: "Netlink Socket: User-Kernel IPC"
date: 2017-05-13 09:50
comments: true
published: false
categories: ipc kernel modules
---

To interact with kernel space from user space, there are several mechanisms that one can use, which includes system calls, ioctl, filesystem (procfs, sysfs) and **netlink socket**. 

Netlink socket is an asynchronous queue based communication channel between kernel space and user space. One can use it to pass information back and forth to kernel modules. Netlink uses the address family **AF_NETLINK** as compared to **AF_INET** used by TCP/IP socket.

In Netlink, communication can be initiated from either side and kernel component of netlink it has not compile time dependencies with loadable modules.

Netlink is used by some popular tools to interact with their kernel space counterparts. For example: **NETLINK_NFLOG** (iptables and netfilter module), **NETLINK_ARPD** (managing arp table from user space).

# Netlink Socket APIs

Netlink uses standard socket, sendmsg, recvmsg and close apis.

