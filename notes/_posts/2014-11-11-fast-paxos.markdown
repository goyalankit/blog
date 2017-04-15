---
layout: post
title: "Fast Paxos"
date: 2014-11-11 15:10:06 -0600
comments: true
categories: 
---

- Algorithm executes multiple rounds.
- An acceptor votes to accept atmost one value in a single round.
- Goal: Achieve consistency by ensuring that different values are not chosen in different rounds.
- Note that rounds can run concurrently, may be skipped altogether.


## Accepter

State maintained by accepter:

{% highlight sh %}
# state maintained by acceptor
rnd[a]
vrand[a]
vval[a]
{% endhighlight %}

- **rnd[a]** - highest numbered round in which a participated, initially 0.
- **vrand[a]** - highest numbered round in which a has cast a vote, initially 0.
- **vval[a]**- The value a voted to accept in round **vrand[a]**; it's initial value is irrelevant. can be inferred using **vrand[a] == 0**

Constraint: **vrand[a] <= rnd[a]** is always true. Since it cannot vote without participating in it. But it can participate and doesn't vote.

Questions?
- what is this value that we are talking about?

## Coordinator (role is often played by acceptors)

- For each round *i*, some coordiantor is pre-assigned to be the coordinator of round *i*.
- Moreover, each coordinator is assigned to be the coordinator for infinitely many rounds.
- __Proposers__ send their proposals to coordinators.
- The coordinator picks a value in round _i_, that it tries to get chosen in that round.

{% highlight sh %}
# State maintined by coordinator:
crnd[c]
cval[c]
{% endhighlight %}

- **crnd[c]** The highest-numbered round c has begun.
- **cval[c]** The value that c has picked for that round or the special value none if c has not yet picked a value for that round.

Questions?
- How do we decide coordinator for a given round *i*?

# The Basic Algorithm

## Phase 1a:
- Coordinator sends a message ( **crnd**, **cval** ) to each acceptor to **participate** in round i.

## Phase 1b:
- Acceptor replies or ignores the message based on following conditions.

{% highlight sh %}
Success:
i > rnd[a]

Fail:
i <= rnd[a] (i.e., a has already begun round i or higher)
{% endhighlight %}

 If success, it sends a message to coordinator containing (**vrand[a]**, **vval[a]**)

## Phase 2a:
- Coordinator, on getting majority messages, sends acceptors a message to vote on __value__(cval, chosen by looking at the contents of Phase 1b message) in round **i**.

## Phase 2b:
- Acceptor accepts the message to vote on v in round i based on following conditions.


{% highlight sh %}
Success:
i >= rnd[a] and vrnd[a] != i

Fail:
i < rnd[a] (accepted has started higher round)
vrand[a] == i (alreaded voted in this round and only one value can be
accepted in this round)
{% endhighlight %}

Note: acceptor ignores the message no fail. On Accept, it sends a message to all **learners** announcing its round i vote.

## Learner
- A learner accepts a value only if it receives message (Phase 2b) from majority of acceptors.

## Important Constraint
* Coordinators are assigned unique rounds, which ensures that phase 2a messages cannot be sent for with different values in same round.


<style>
.paxos_basic img {
  width: 600px;
}
</style>
{:.paxos_basic}
![Paxos Basic](http://i.imgur.com/f6kKDsL.jpg?1)


## Picking a value in phase 2a

**CP**: For any round i and j with j < i, if a value v has been chosen or might yet be chosen in round j, then no acceptor can vote for any value except v in round i.

---

* paper doesn't discuss leader election (coordinator)

