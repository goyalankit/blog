---
layout: post
title: "Note on hypervisors: Type 1 vs Type 2"
date: 2017-04-15 12:33
comments: true
categories: hello
---

# Hypervisors

## Type 1 hypervisor
Type 1 hypervisor means that vm is running natively (bare metal) on hardware.

Examples: VMWare vSphere / ESXi, KVM, Xen / Citrix XenServer

## Type 2 hypervisor
Type 2 hypervisor means that vm is running on another peice of software. Basically on a host OS. 

Example: VMWare workstation.

# QEMU, KVM and QEMU with KVM

<style>
.make_point {
  color: rgba(36, 40, 41, 0.79);
}
</style>
{:.make_point}
**QEMU is a type 2 hypervisor but it can become Type 1 if used with KVM.**

Here's how:

Virtualization technologies like VT-x (intel) and AMD-V, **map physical CPU directly to Virtual CPU.**

**KVM enables mapping of physical cpu to virtual cpu**. This provides hardware acceleration for virtual machine.

QEMU can use KVM (Linux kernel module) as vert-type to take advantage of this hardware acceleration and thus can become type 1.

QEMU used without KVM, can use TCG (Tiny co  de generator); if your cpu doesn’t support hardware virtualization. Then it’s a type 2 hypervisaor.

[Further Reading](http://www.innervoice.in/blogs/2014/03/10/kvm-and-qemu/)
