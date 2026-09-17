---
layout: page
title: DNS64 with Bind
parent: DNS64
nav_order: 1
------------

# DNS64 with Bind

Edit:

```bash
nano /etc/bind/named.conf.options
```

Inside the `options` section, add:

```text
dns64 64:ff9b::/96 {
    clients { any; };
};
```

Check the configuration:

```bash
named-checkconf
```

If no errors are reported, reload BIND:

```bash
rndc reload
```

# Test DNS64

```bash
dig @localhost ipv4only.%DOMAIN% AAAA
```

Yyou should receive an IPv6 address beginning with `64:ff9b::`.

Where does the AAAA record come from?

The answer is: the resolver created it.

# Is the AAAA record really in DNS?

Let's verify that the authoritative server does not contain the AAAA record.

Query your authoritative server directly:

```bash
dig @ns.%DOMAIN% ipv4only.%DOMAIN% A
dig @ns.%DOMAIN% ipv4only.%DOMAIN% AAAA
```

The authoritative server has the A record but no AAAA record.

This is why we call the DNS64 answer a **synthetic AAAA record**.

DNS64 does not modify the authoritative DNS zone.

The synthetic record is created by the recursive resolver while answering the client's query.
