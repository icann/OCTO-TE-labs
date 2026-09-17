---
layout: page
title: DNS64 with Bind
parent: DNS64
nav_order: 1
---

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

You should receive an IPv6 address beginning with `64:ff9b::`.

Where does the AAAA record come from?

The answer is: the resolver created it.
