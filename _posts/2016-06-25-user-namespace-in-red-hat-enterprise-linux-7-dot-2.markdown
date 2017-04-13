---
layout: post
title: "User Namespace in Red Hat Enterprise Linux 7.2"
date: 2016-06-25 15:34
comments: true
published: true
categories: rhel7.2 namespace containers 
---


Red Hat announced the availability of [user namespace](https://lwn.net/Articles/532593/) in RHEL 7.2
[release notes](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html-single/7.2_Release_Notes/index.html#technology-preview-kernel),
but they don't give details on how to use them. By default in RHEL 7.2, user
namespaces are disabled.



## Verify if user namespace is enabled
You can run a quick check by executing the
[demo_userns.c](https://lwn.net/Articles/539941/) program, that creates a child
in new user namespace. The child simply prints its effective user, groupd IDs
and capabilities. If it runs successfuly, then namespaces are already enabled
for you. However, if it returns something like `clone: Invalid argument`, **then user
namespaces are disabled.**

You might need to install following libraries to run the *demo_userns.c*:

{% highlight sh %}
sudo yum install libcap-devel
{% endhighlight %}

Compile it using lcap:

{% highlight sh %}
gcc -lcap demo_ns.c -o demo_ns
{% endhighlight %}



## Enable user namespace
To enable user namespace, you need to change one of the kernel parameters. You
can do it by running following command:


{% highlight sh %}
sudo grubby --args="user_namespace.enable=1" \
  --update-kernel=/boot/vmlinuz-3.10.0-327.el7.x86_64
{% endhighlight %}

Note: you might need to change the version of `vmlinuz` executable.

**Reboot the box.**

*Now you can verify by running [demo_userns.c](https://lwn.net/Articles/539941/)
again and it should print user id, group id and capabilities*

{% highlight sh %}
eUID = 65534;  eGID = 65534;  capabilities: = cap_chown,cap_dac_override,
cap_dac_read_search, cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,
...
{% endhighlight %}

Till then.

