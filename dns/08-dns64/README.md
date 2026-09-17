---
layout: page
title: DNS64
parent: DNS
nav_order: 8
------------

# DNS64

IPv6 only clients need a way to reach the IPv4 internet. 
DNS64 and NAT64 solve this problem.

- DNS64 creates a synthetic IPv6 (AAAA) record from an IPv4 (A) record.
- NAT64 translates the actual network traffic between IPv6 and IPv4.

DNS64 does **not** translate network packets.

Before configuring anything, let's examine how DNS behaves without DNS64.

# IPv4 and IPv6 DNS records

DNS uses different record types for IPv4 and IPv6 addresses:

```text
A       IPv4 address
AAAA    IPv6 address
```

A host may have only an IPv4 address, only an IPv6 address, or both.

In our lab we have hosts that allow us to test these different cases.

First query the IPv4-only host:

```bash
dig ipv4only.%DOMAIN% A
dig ipv4only.%DOMAIN% AAAA
```

What do you see?

Now query the IPv6-only host:

```bash
dig ipv6only.%DOMAIN% A
dig ipv6only.%DOMAIN% AAAA
```

This time you should find a AAAA record but no A record.

# The IPv6-only problem

Imagine an IPv6-only client that wants to connect to:

```text
ipv4only.%DOMAIN%
```

The application needs an IPv6 destination address.

It therefore asks DNS for:

```text
ipv4only.%DOMAIN% AAAA
```

But the server only has an IPv4 address.

There is no AAAA record.

Try:

```bash
curl -6 http://ipv4only.%DOMAIN%
```

Does it work?

Now compare this with:

```bash
curl -4 http://ipv4only.%DOMAIN%
```

We need a mechanism that allows an IPv6-only application to reach this IPv4-only server.

This is where DNS64 and NAT64 are used.

# NAT64

Our network has a NAT64 translator.

NAT64 translates packets between IPv6 and IPv4.

The well-known IPv6 prefix for IPv4/IPv6 translation is:

```text
64:ff9b::/96
```

The last 32 bits contain an IPv4 address.

For example, the IPv4 address:

```text
192.0.2.1
```

can be represented inside the NAT64 prefix as:

```text
64:ff9b::192.0.2.1
```

The same address can also be written using hexadecimal IPv6 notation.

```text
64:ff9b::c000:201
```

The NAT64 translator extracts the IPv4 address from the IPv6 destination address and translates the packet to IPv4.

But there is still a problem.

How does an application know which NAT64 IPv6 address to use?

That is the job of DNS64.

# Test NAT64 directly

Now let's test the NAT64 translator independently of DNS.

First find the IPv4 address of the IPv4-only server:

```bash
dig ipv4only.%DOMAIN% A
```

Take the IPv4 address from the answer.

Now use that IPv4 address directly with the NAT64 prefix:

```bash
curl -6 http://[64:ff9b::%IPv4%]
```

DNS64 was not involved in this test.

You constructed the NAT64 destination address yourself.

This demonstrates an important distinction:

```text
DNS64
    creates an IPv6 destination address

NAT64
    translates packets sent to that address
```

# DNS64

DNS64 is implemented by a recursive DNS resolver.

When a client asks for an AAAA record and the destination has an IPv4 address but no IPv6 address, the DNS64 resolver can create a **synthetic AAAA record**.

In our lab:

```text
ipv4only.%DOMAIN%.    IN A    %IPv4%```

but no AAAA record.

A DNS64 resolver can instead construct:

```text
64:ff9b::%IPv4%
```

and return this as a synthetic AAAA record.

The client can then connect to this IPv6 address.

The packets are sent to the NAT64 translator, which translates them to IPv4.

# Establish a baseline

Before enabling DNS64, let's compare our two recursive resolvers.

Query `resolv1`:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% A
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA
```

Now query `resolv2`:

```bash
dig @100.100.%GRP%.68 ipv4only.%DOMAIN% A
dig @100.100.%GRP%.68 ipv4only.%DOMAIN% AAAA
```

The answers from both resolvers should currently be the same.

Both resolvers can find the IPv4 A record.

Neither should return an IPv6 address for the AAAA query.

Remember these results.

Please enable now DNS64 on resolv1 and resolv2:

- [DNS64 with Bind](bind.md)
- [DNS64 with Unbound](unbound.md)


# Test DNS64 and NAT64 together

Now let the application use DNS normally.

Make sure your client is using `resolv1` as its recursive resolver.

Run:

```bash
curl -6 http://ipv4only.%DOMAIN%
```

This time you supplied only a hostname.

What happened?

Two completely different systems participated:

1. DNS64 created the IPv6 address.
2. NAT64 translated the network traffic.

# Compare native IPv6 with NAT64

Now try:

```bash
curl -6 http://ipv6only.%DOMAIN%
```

This connection should use native IPv6.

Check the DNS response:

```bash
dig ipv6only.%DOMAIN% AAAA
```

Does the IPv6 address begin with `64:ff9b::`?

It should not.

Compare this with:

```bash
dig ipv4only.%DOMAIN% AAAA
```

The first destination is reached using native IPv6.

The second destination is reached through NAT64.

From the application's point of view, however, both connections use IPv6.

# DNS64 and DNSSEC

DNSSEC introduces an interesting complication.

The authoritative DNS server can cryptographically sign the DNS records it publishes.

But the synthetic AAAA record created by DNS64 does not exist in the authoritative zone.

The authoritative server therefore cannot have signed that synthetic AAAA record.

Try:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA +dnssec
```

Think about the following question:

> If DNSSEC is intended to verify that DNS data came unchanged from the authoritative zone, how can a resolver return a DNS64-generated AAAA record that was never present in that zone?

DNS64 and DNSSEC therefore require special handling by validating resolvers.

We will investigate DNSSEC in more detail in the DNSSEC labs.

# Final tests

Test IPv4 connectivity:

```bash
curl -4 http://ipv4only.%DOMAIN%
```

Test native IPv6 connectivity:

```bash
curl -6 http://ipv6only.%DOMAIN%
```

Test NAT64 directly:

```bash
curl -6 http://[64:ff9b::%IPv4%]
```

And finally test DNS64 and NAT64 together:

```bash
curl -6 http://ipv4only.%DOMAIN%
```

All four tests should work, but they do not all use the same mechanism.

Make sure you understand why.

# Questions

Before continuing, make sure you can answer the following questions:

1. What is the difference between an A and an AAAA record?

2. Why can't an IPv6-only application normally connect to a host that has only an A record?

3. Where is a DNS64 synthetic AAAA record created?

4. Does DNS64 add the synthetic AAAA record to the authoritative DNS zone?

5. What is `64:ff9b::/96`?

6. Which part of a synthesized `64:ff9b::/96` address contains the IPv4 destination address?

7. What does DNS64 do?

8. What does NAT64 do?

9. Does DNS64 translate any network packets?

10. What happens when a hostname already has a usable AAAA record?

11. Why did `resolv1` and `resolv2` initially give different answers for the same AAAA query?

12. Why can `curl -6 http://ipv4only.%DOMAIN%` work even though the web server itself has no IPv6 address?

13. When you use:

    ```bash
    curl -6 http://[64:ff9b::%IPv4%]
    ```

    is DNS64 involved?

14. When you use:

    ```bash
    curl -6 http://ipv4only.%DOMAIN%
    ```

    which roles are played by DNS64 and NAT64?

# Summary

DNS64 and NAT64 allow IPv6-only clients to communicate with IPv4-only servers.

DNS64 creates a synthetic AAAA record when a destination has an IPv4 address but no usable IPv6 address.

NAT64 operates on network packets.

Together they allow an IPv6 application to use a normal hostname:

Please see also the Best Current Practice [RFC 10001](https://datatracker.ietf.org/doc/rfc10001).
