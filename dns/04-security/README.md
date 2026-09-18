---
layout: page
title: "Security"
parent: "DNS"
nav_order: 4
---

# Name Server Security

Anything connected to the internet is subjected to a constant barrage of hacking or abuse attempts.
For new wordpress sites it takes on average below 10 seconds for the first attack to arrive.
DNS servers are not long behind. Therefore it is of critical importance to harden your servers before
they go online.

## Hardening the OS

First step in any security for applications is a secure operating system. Security is build in layers and 
if the foundations aren't safe, it is very hard (or impossible) to mitigate that.

Hardening the OS, starts with making sure that no unneccessary ports are open, ssh only accepts public key authentication,
only users can login (no remote root access) and many other security best practices. Hopefully you or your
company already have routines in place for hardening your machines.

Hardening the OS is beyond this course, but it is the foundation of all further steps.

## Hardening Authoritative DNS Servers

The DNS has many attack vectors, from spoofing to DDoS and beyond. But why spoof a response 
when you easily can change data on the autoritative server? So hardening and protecting your 
DNS infrastructure is important for the security of your IT infrastructure. 
DNS is used to secure email, all certificates and even many firewalls use DNS based rules.

Please find answers to these questions:
- Can another group query us?
- Can another group recurse through us?
- Can a ns1 and ns2 transfer the zone?
- Can anyone else request AXFR?
- Can anyone else send NOTIFY?

Please folow the labs:
  - [Primary Security with Bind](primarybind.md)
  - [Secondary Security with Bind](secondarybind.md)
  - [Secondary Security with NSD](secondarynsd.md)

Have the answers changed?
- Can another group query us?
- Can another group recurse through us?
- Can a ns1 and ns2 transfer the zone?
- Can anyone else request AXFR?
- Can anyone else send NOTIFY?

## Hardening Resolvers

Resolvers can be attacked themselfs or be used to attack others.
It is a good idea to prevent both. 

Please follow theses labs to secure your lab infrastructure. 

> [!TIP] 
> These lab instructions are a little bit more complex to follow.
> You need to execute each step according to the software that you have installed.
> Please finish each step for the different DNS server software you have installed 
> before proceeding to the next step.

