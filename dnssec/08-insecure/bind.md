---
layout: page
title: "Bind"
parent: "Going Insecure"
nav_order: 1
---

# Going Insecure

Sometimes it unfortunately becomes neccessary to switch off DNSSEC, with other words "going insecure".
It  might seems that this is an easy operation, and in many ways it is. But a few precautions have to be taken.

Due to all the caches on the internet, it is not possible to just switch off DNSSEC. It has be phased out. The correct order of steps is

1. Remove DS record from parent
1. Wait two times maximum of TTL of DS record
1. Switch off DNSSEC on primary server

To ease this process Bind has a special DNSSEC policy. Just change your configuration to 

```
dnssec-policy "insecure";
```
Again watch the DNSSEC configuration with
```
sudo rndc dnssec -status grp%GRP%.%DOMAIN%
```
and watch the parent remove the DS record with
```
dig grp%GRP%.%DOMAIN% DS +dnssec +noall +answer
```
