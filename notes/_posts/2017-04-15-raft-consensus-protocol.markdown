---
layout: post
title: "Raft - consensus protocol"
date: 2017-04-15 11:25
comments: true
categories: raft distributed systems
---
# What is consensus?

In short concensus is agreement on shared state (Single system image). It's typically used to build replicated state machine.

**Replicated log** === **replicated state machine.**

Idea is that if the log on all different servers looks the same, the state machine looks the same.
All the state machines produce the same result for the same set of operations.

## What happens in case of failures
Recovers from server failures autonomously.

* **minority fail**: no problem.
* **majority fail**: lose availability, retain consistency.


## Failure Model
Raft consider only fail-stop (not Byzantine) model, delayed/lost messages.

## How is consensus used?
Consensus can be used to replicate whole databases like Spanner from Google. It can
be used to maintain system level configuration like etcd, nomad, etc.

# Raft Overview

Basic concept is to Parition your data and provide each partition to a state machine. Raft heavily relies
on leader which is an after thought in Paxos.

## Leader Election
Select one of the servers as leader, detect crashes and choose a new leader.

## Log Replication
Leader takes commands from clients, append those to leader log and leader replicates its log to other servers.

## Safety
Only server with up-to-date log can become leader.

# Approaches to Consensus:

1. **Symmetric, leader-less**
* All servers have equal roles
* clients can connect to any server

2. **Asymmetric, leader based**
* At any given time, one server is in charge, others accept its decisions.
* Clients communicate with leader.


**Raft uses leader, all the complexity comes in when leader fails**

# Leader Election

1. **Safety**: allow at most one winner per term.
* each server has only one vote
* two different servers cannot accumulate majority.

2. **Liveness**: some candidate must eventually win.
* choose elections timeouts randomly.

# Log

- Each servers has its own copies of logs.
- Each log in composed of entries and each entry has an index number.
- Each entry contains a command and a term number (increases monotonically.)

**Committed**: If a particular entry is stored on majority of server, we say that entry is committed. {Not Really}


# Normal Operation:
- Client sends command to leader
- Leader appends the command to its log
- Leader sends AppendEntries RPCs to followers
- Once new entry committed:
   - Leader passes command to state machine, reruts result to client.
   - Leader notifier followers of commited enteries in subsequent AppendEntries RPCs
   - Followers pass committed commands to their state machines.
- Need only majority to respond that command is stored.
- Slow machines don’t slow down the clients.

# Log Consistency
**Properties that are always true**
1. Combination of index and term uniquely idenfies a long entry. i.e., they will have the same comand. Also, the logs are identical in all preceding entries.

2. If a given entry is committed, all preceding entries are also committed.

Above properties are enforced by check made during **ApendEntries** consistency check.

It includes two values: Index and term of entry just before the new one.

If the log doesn’t have that entry, RPC will be rejected. It's kind of like an induction step. Which proves that if an entry in a server matches a leader entry, then all preceding entries also match.

# Leader Changes
- Old leader may have left entries partially replicated.
- No special step taken when new leader comes up. Continues normal operation.
cleanup has to happend during the normal operation.

## Raft Approach:
— Assume that leader’s log is always correct.
Will eventually make follower’s logs identical to leader’s.

Scenario slide 15:
S4,S5 were leaders for term 2,3 and 4.
Safety Requirement:
Once a log entry has been applied to a state machine, no other state machine must apply a different value for that log entry.

Raft Guarantee:
Once a leader has decided that a particular entry is committed, that entry will be present in the logs of all future leaders.

How?
- Leader will never overwrite its own entry.
- Onlt enteries in leaders log can be committed.
- Entry must be committed before applying to state machine.

    COMMITTED                         —>       Present in future leaders’ logs
(Restriction on committment)                   (Restrictions on leader election)
Picking the best Leader

We can’t know for sure if entry is committed.
We elect the server that has the log that is most likely all the enteries that are committed.

When candidate request a vote: it includes:
index and term of last log entry. -> unqiuely entire log.

1. If voters log is more complete, deny the vote.
2. If the terms match and the voting servers log is longer, deny the vote.

Whoever wins the election is guarantted to have the most complete log among the candidate servers.


Explanation of [slides](https://raft.github.io/slides/raftuserstudy2013.pdf) from Raft user study:

Slice 18:

Interesting cases to consider:

* Entry being committed is in current term
* Most recent entry is declared committed.
* S4 or S5 cannot become leaders if S1 is down.

Slide 19:

* If S1 is down, S2/S3 will become leader.
* Cannot consider entry 2 to be committed.

**Rules for committment**:
* Must be stored on a majority of servers
* At least one new entry from leader’s term must also be stored on majority of servers. Combination of election rules and commitment rules makes Raft safe.

# Reparing Logs

Slide 21:
- Followers missing entries (a, b, e)
- Extra enteries (d, f, c)
- Get rid of all extra eneteries and fill them with enteries from leaders log.
- Leader keeps nextIndex for each follower, nextIndex is 1 + leader’s last index.

On receiving AppendEnteries, consistency check fails, decrement the nextIndex and try again.
If follower overwrites inconsistent entry, it deletes all subsequent entries.
Old leader may not be dead

- Temporarily disconnected from network.

# Client Protocol

## Sender commands to leader
— If leader unknown, contact any server
— If contacted server not leader, it will redirect to leader.

Leader doesn’t response until logged, committed and executed by leader’s state machine.
- If request times out, client reissues command to some other server.
- eventulaly executed -> might get executed twice.

Client embeds unique id in each command. Before accepting command, leader checks its log for entry with that id.

## Linearizability: exactly once semantics.
Configuration Changes:
System configuration:
- ID, address for each server.
- Determines what constitutes a majority.

Consensus mechanism must support changes in the configuration:
- Replace failed machine.
- Change degree of replication.
Cannot switch directly from one configuration to antoher: conflictin majorities could arise.
— conflicting majorities

## Two Phase:

**1st phase:** 
joint consensus (intermediate phase that need majority of both old and new configurations for elections.)
union of old and new configurations. separate 

**2nd phase:**
clients makes a request to leader, server adds an entry to log describing new configuration. 
- propogates to another servers.
- configuration changes takes effect immediately.


