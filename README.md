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

# Lab environment

Every participant in the lab gets their own lab environment

Your instructor will announce the ***lab domain*** and give you a ***group number***.

> [!IMPORTANT]
>
> In all this lab, be carefull to always replace ***X*** by your group
> number in IP addresses, server name and any other place where
> required. Same for \<***lab domain***\> to be replace by the 
> domain name registered for the class.

Your domain: grp***X***.\<***lab domain***\>

Every participant gets a number of virtual machines. The machines are placed in different networks.

![network topology](img/topology.png)

| Device Name   | IPv4 Address          | IPv6 Address                | Network  |
| ------------- | --------------------- | --------------------------- | -------- |
| cli           | 100.100.***X***.2     | fd73:7c99:***X***::2        | lan      |
| resolv1       | 100.100.***X***.67    | fd73:7c99:***X***:64::67    | internal |
| resolv2       | 100.100.***X***.68    | fd73:7c99:***X***:64::68    | internal |
| soa           | 100.100.***X***.66    | fd73:7c99:***X***:64::66    | internal |
| ns1           | 100.100.***X***.130   | fd73:7c99:***X***:128::130  | external |
| ns2           | 100.100.***X***.131   | fd73:7c99:***X***:128::131  | external |

# The Hands-On Labs

Please don't skip any of the labs as they build on each other. 

> [!WARNING]
> Later labs will likely fail if you didn't finish the earlier ones!

- [DNS 01 - DNS Tools](DNS%2001%20-%20DNS%20Tools.md)
- [DNS 01a - Dig](DNS%2001a%20-%20Dig.md)
- [DNS 01b - EDE](DNS%2001b%20-%20EDE.md)
- [DNS 01c - Cookies](DNS%2001c%20-Cookies%20.md)
- [DNS 01d - DNSviz](DNS%2001d%20-%20DNSviz.md)
- [DNS 01e - Zonemaster](DNS%2001e%20-%20Zonemaster.md)
- [DNS 02 - Authoritative](DNS%2002%20-%20Authoritative.md)
- [DNS 02a - Primary Bind](DNS%2002a%20-%20Primary%20Bind.md)
- [DNS 02a - Primary Knot](DNS%2002a%20-%20Primary%20Knot.md)
- [DNS 02a - Primary PowerDNS](DNS%2002a%20-%20Primary%20PowerDNS.md)
- [DNS 02b - Secondary Bind](DNS%2002b%20-%20Secondary%20Bind.md)
- [DNS 02b - Secondary NSD](DNS%200b%20-%20.Secondary%20NSDmd)
- [DNS 02b - Secondary Knot](DNS%2002b%20-%20Secondary%20Knot.md)
- [DNS 02b - Secondary PowerDNS](DNS%2002b%20-%20Secondary%20PowerDNS.md)
- [DNS 03 - Reverse](DNS%2003%20-%20Reverse.md)
- [DNS 04 - Resolvers](DNS%2004%20-%20Resolver.md)
- [DNS 05 - Security](DNS%2005%20-%20Security.md)
- [DNS 05a - Authoritative Security](DNS%2005a%20-%20Authoritative%20Security.md)
- [DNS 05b - Resolver Security](DNS%2005b%20-%20Resolver%20Security.md)
- [DNSSEC 01 - Signing](DNSSEC%2001%20-%20Signing.md)
- [DNSSEC 02 - Chain of Trust](DNSSEC%2002%20-%20Chain%20of%20Trust.md)
- [DNSSEC 03 - Validation](DNSSEC%2003%20-%20Validation.md)
- [DNSSEC 04 - Key Rollover](DNSSEC%2004%20-%20Key%20Rollover.md)
- [DNSSEC 05 - Automated DNSSEC](DNSSEC%2005%20-%20Automated%20DNSSEC.md)
- [DNSSEC 06 - ZONEMD](DNSSEC%2006%20-%20ZONEMD.md)
