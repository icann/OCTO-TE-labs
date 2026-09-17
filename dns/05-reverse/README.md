---
layout: page
title: "Reverse DNS"
parent: "DNS"
nav_order: 5
---

# Reverse DNS

Everybody knows that the DNS can be used to translate names to IP addresses. But DNS can also translate IP addresses to names. This is called **Reverse DNS**.

For IPv4, reverse DNS uses the special domain `in-addr.arpa` and for IPv6 it uses `ip6.arpa`.

For example, to find the name associated with `192.0.43.7` you have to query for a PTR record for and the name `7.43.0.192.in-addr.arpa`. For the IPv6 address `2001:500:88:200::8` it is till a PTR record
but the name is `8.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.8.8.0.0.0.0.5.0.1.0.0.2.ip6.arpa`.

Notice that the octets of the IP address appear in reverse order and that IPv6 addresses a fully expanded.

The `dig` command can construct this name automatically:

```
dig -x 192.0.43.7
```
or 
```
dig -x 2001:500:88:200::8
```

Please follow the labs.
