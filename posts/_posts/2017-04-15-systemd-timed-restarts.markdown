---
layout: post
title: "Timed systemd service restarts"
date: 2017-04-15 12:33
comments: true
categories: hello
---

If you have ever wanted to do periodic restarts of a service, and for whatever
reason you don't want to use cron, you can use systemd to accomplish that task.

Systemd has numerous knobs and you can do this task in several different ways.
Here's one that I got to work for me:

# Concept
Create a two hour timer which activates a target every two hours. Our target
is a one-shot service which restarts the main service.

Here's the code ([gist](https://gist.github.com/goyalankit/e8223915c382b98cfe99fff57bbc52dc)):

<style>
.filename strong {
  color: rgba(4, 4, 181, 0.7);
}
</style>
{:.filename}
**two-hour.timer**

{% highlight sh %}

[Unit]
Description=2hour timer

[Timer]
OnBootSec=0min
OnCalendar=0/2:00:00
Unit=two-hour.service

[Install]
WantedBy=basic.target

{% endhighlight %}


{:.filename}
**two-hour.service**

{% highlight sh %}

[Unit]
Description=Service that restarts my spread_goodness.service every two hours.

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl try-restart spread_goodness.service

{% endhighlight %}

You can check the current state of timers using `systtemctl`.

{% highlight sh %}
# Check that timers are active.
$ systemctl list-timers

NEXT                         LEFT          LAST                         \
Mon 2017-04-15 22:00:00 UTC  1h 52min left Mon 2017-04-10 20:13:00 UTC  \

PASSED               UNIT                                  ACTIVATES
4min 49s ago         two-hour.timer                        two-hour.service

{% endhighlight %}

Goodbye.
