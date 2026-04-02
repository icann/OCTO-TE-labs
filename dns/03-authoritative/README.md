---
layout: page
title: "Authoritative DNS"
parent: "DNS"
nav_order: 3
---

# Configuring authoritative name servers

We are going to build a "hidden primary" setup, where the SOA server is the hidden primary and NS1 and NS2
will server the zone externaly.

> [!IMPORTANT]
> "Hidden primary" is a very important DNS concept! 
> If you haven't heard of or unsure about the 
> configuration, please ask your instructor. 

## From the parent zone

Our "parent" (***%DOMAIN%***) has already created the following in its own zone:

```
grp%GRP%             NS          %DOMAIN%.
```

The lab uses dnsdist (a DNS proxy) to forward queries to the 
respective groups ns1 and ns2 servers. It expects these server at the
following ip addresses:

| Device Name   | IPv4 Address   | IPv6 Address         | 
| ------------- | -------------- | -------------------- |
| ns1           | 100.100.%GRP%.130  | %IPv6pfx%:%GRP:128.::130 |
| ns2           | 100.100.%GRP%.131  | %IPv6pfx%:%GRP:128.::131 |

Our zone configuration must be compatible with that.

## Setting up the primary

Use the "SOA" server as primary authoritative server for the  grp%GRP%.***%DOMAIN%*** zone.

Your instructor will tell you which instructions to follow for installation of your primary server.

- [Primary Setup with Bind](primarybind.md) 

## Setting up the secondaries

Your instructor will tell you which instructions to follow for installation of your secondary servers.

- [Secondary Setup with Bind](secondarybind.md)
- [Secondary Setup with NSD](secondarynsd.md)

Once you are done with the configuration of your primary and secondary servers, please come back here and continue with the next section!

## Test your zone configuration and propagation.

We will now use *dig* tool to verify the zone configuration and propagation, then do the same for one or two other groups in the class and share comments. From your client, run the following dig queries. All should return **answer section** otherwise you must review your configurations before continuing:

On the **cli** instance

1. `dig @100.100.%GRP%.66  grp%GRP%.%DOMAIN% SOA`
1. `dig @100.100.%GRP%.130 grp%GRP%.%DOMAIN% SOA`
1. `dig @100.100.%GRP%.131 grp%GRP%.%DOMAIN% SOA`

Please repeat the following queries several times

```dig @%DOMAIN% grp%GRP%.%DOMAIN% SOA        +nsid```



