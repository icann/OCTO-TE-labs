[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

# Technical Engagement DNS Lab

Welcome to the DNS lab of ICANNs Technical Engagement Team!

To participate in this lab you will need 

- to bring your own laptop
- some DNS knowledge
- some experience with the Linux command line

> [!TIP] 
> Bring a good mood with you. :grin:

The lab consists of a number of presentations that will explain
parts of the DNS ecosystem and a number of practical hands-on labs.

# License

This repository is licensed under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0)

For details please see [License.md](License.md)

# Lab environment

Every participant in the lab gets their own lab environment

Your instructor will announce the ***lab domain*** and give you a ***group number***.

> [!IMPORTANT]
>
> In all this lab, be carefull to always replace ***X*** by your group
> number in IP addresses, server name and any other place where
> required. ***lab domain*** is to be replaced by the 
> domain name registered for the class.

Your domain: grpX.lab_domain

Every participant gets a number of virtual machines. The machines are placed in different networks.

We use the private 100.64.0.0/10 address space from [RFC 6598](https://www.rfc-editor.org/rfc/rfc6598).

![network topology](img/topology.png)

| Device Name   | IPv4 Address          | IPv6 Address                | Network  |
| ------------- | --------------------- | --------------------------- | -------- |
| cli           | 100.100.***X***.2     | fd89:59e0:***X***::2        | lan      |
| resolv1       | 100.100.***X***.67    | fd89:59e0:***X***:64::67    | internal |
| resolv2       | 100.100.***X***.68    | fd89:59e0:***X***:64::68    | internal |
| soa           | 100.100.***X***.66    | fd89:59e0:***X***:64::66    | internal |
| ns1           | 100.100.***X***.130   | fd89:59e0:***X***:128::130  | external |
| ns2           | 100.100.***X***.131   | fd89:59e0:***X***:128::131  | external |

# Here be Dragons

Please don't skip any of the labs as they build on each other. 

> [!WARNING]
> Later labs will likely fail if you didn't finish the earlier ones!

> [!WARNING]
> Any all configurations in this lab are not ready for production systems.

# The Hands-On Labs

- [DNS 01 - Resolving](DNS%2001%20-%20Resolving.md)
    - [DNS 01a - Resolving with Bind](DNS%2001a%20-%20Resolving%20with%20Bind.md)
    - [DNS 01b - Resolving with Unbound](DNS%2001b%20-%20Resolving%20with%20Unbound.md)
- [DNS 02 - DNS Tools](DNS%2002%20-%20DNS%20Tools.md)
    - [DNS 02a - Dig](DNS%2002a%20-%20Dig.md)
    - [DNS 02b - EDE](DNS%2002b%20-%20EDE.md)
    - [DNS 02c - DNSviz](DNS%2002c%20-%20DNSviz.md)
    - [DNS 02d - Zonemaster](DNS%2002d%20-%20Zonemaster.md)
    - [DNS 02e - Other Tools](DNS%2002e%20-%20Other%20Tools.md)
- [DNS 03 - Authoritative](DNS%2003%20-%20Authoritative.md)
    - [DNS 03a - Primary Bind](DNS%2003a%20-%20Primary%20Bind.md)
    - [DNS 03b - Secondary Bind](DNS%2003b%20-%20Secondary%20Bind.md)
    - [DNS 03b - Secondary NSD](DNS%2003b%20-%20Secondary%20NSD.md)
- [DNS 04 - Security](DNS%2004%20-%20Security.md)
    - [DNS 04a - Authoritative Security](DNS%2004a%20-%20Authoritative%20Security.md)
    - [DNS 04b - Resolver Security](DNS%2004b%20-%20Resolver%20Security.md)
- [DNS 05 - Reverse DNS](DNS%2005%20-%20Reverse%20DNS.md)
- [DNS 06 - RPZ](DNS%2006%20-%20RPZ.md)
    - [DNS 06a - RPZ with Bind](DNS%2006a%20-%20RPZ%20with%20Bind.md)
    - [DNS 06a - RPZ with Unbound](DNS%2006a%20-%20RPZ%20with%20Unbound.md)
- [DNS 07 - Local Root](DNS%2007%20-%20Local%20Root.md)
    - [DNS 07a - Local Root with Bind](DNS%2007a%20-%20Local%20Root%20with%20Bind.md)
    - [DNS 07a - Local Root with Unbound](DNS%2007a%20-%20Local%20Root%20with%20Unbound.md)
---
- [DNSSEC 01 - Dig DNSSEC](DNSSEC%2001%20-%20Dig%20DNSSEC.md)
- [DNSSEC 02 - Validation](DNSSEC%2002%20-%20Validation.md)
    - [DNSSEC 02a - Validation](DNSSEC%2002a%20-%20Validation%20with%20Bind.md)
    - [DNSSEC 02a - Validation](DNSSEC%2002a%20-%20Validation%20with%20Unbound.md)
- [DNSSEC 03 - Manual Signing](DNSSEC%2003%20-%20Manual%20Signing.md)
    - [DNSSEC 03a - Manual Signing with Bind](DNSSEC%2003a%20-%20Manual%20Signing%20with%20Bind.md)
- [DNSSEC 04 - Chain of Trust](DNSSEC%2004%20-%20Chain%20of%20Trust.md)
- [DNSSEC 05 - ZSK Rollover](DNSSEC%2005%20-%20Key%20Rollover.md)
    - [DNSSEC 05a - ZSK Rollover with Bind](DNSSEC%2005%20-%20Key%20Rollover%20with%20Bind.md)
- [DNSSEC 06 - Automatic Signing](DNSSEC%2006%20-%20Signing.md)
    - [DNSSEC 06a - Automatic Signing with Bind](DNSSEC%2006a%20-%20Signing%20with%20Bind.md)
- [DNSSEC 07 - Chain of Trust](DNSSEC%2007%20-%20Chain%20of%20Trust.md)
- [DNSSEC 08 - KSK Rollover](DNSSEC%2008%20-%20KSK%20Rollover.md)
    - [DNSSEC 08a - KSK Rollover with Bind](DNSSEC%2008a%20-%20KSK%20Rollover%20with%20Bind.md)
- [DNSSEC 09 - Automated DNSSEC](DNSSEC%2009%20-%20Automated%20DNSSEC.md)
    - [DNSSEC 09a - Automated DNSSEC with Bind](DNSSEC%2009a%20-%20Automated%20DNSSEC%20with%20Bind.md)
- [DNSSEC 10 - Zone Delivery](DNSSEC%2010%20-%20Zone%20Delivery.md)
- [DNSSEC 11 - Going Insecure](DNSSEC%2011%20-%20Going%20Insecure.md)
    - [DNSSEC 11a - Going Insecure with Bind](DNSSEC%2011a%20-%20Going%20Insecure%20with%Bind.md)

# Roadmap
- ZONEMD
- RPZ
- Catalog Zones
- Pre Delegation Testing
- PowerDNS / Recursor
- Knot DNS / Resolver
