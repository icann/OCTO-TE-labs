---
layout: page
title: DNS64 with Unbound
parent: DNS64
nav_order: 2
---

# DNS64 with Unbound

Edit the Unbound configuration:

```bash
nano /etc/unbound/unbound.conf.d/dns64.conf
```

Modify the module-config statement and add the dns64-prefix statement:

```text
server:
    module-config: "dns64 validator iterator"
    dns64-prefix: 64:ff9b::/96
```

Check the configuration:

```bash
unbound-checkconf
```

Reload Unbound:

```bash
sudo unbound-control reload
```

# Test DNS64

```bash
dig @localhost ipv4only.%DOMAIN% AAAA
```

You should receive an IPv6 address beginning with `64:ff9b::`.

Where does the AAAA record come from?

The answer is: the resolver created it.
