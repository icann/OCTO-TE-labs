---
layout: page
title: "DNS64"
parent: "DNS"
nav_order: 8
---

# DNS64

# Enable DNS64 on resolv1
```
    // Enable DNS64 with the well-known NAT64 prefix
    dns64 64:ff9b::/96 {
        // clients: which source addresses should receive synthesized records
        // Use "any" for all clients, or restrict to specific IPv6 prefixes
        clients { any; };

        // mapped: which IPv4 addresses to synthesize records for
        // Use negated ACL entries to skip ranges such as RFC 1918 space
        mapped {
            !100.64.0.0/10
            any;
        };

        // exclude: IPv6 AAAA records to ignore if they are already present
        // This is commonly used to ignore previously translated AAAA records
        exclude { 
                64:ff9b::/96; 
                ::ffff:0.0.0.0/96; 
        };

        // suffix: optional IPv6 suffix to append (rarely needed)
        // suffix ::;

        // recursive-only: set to yes to synthesize only for recursive queries
        recursive-only yes;
    };
```

# Test
- compare results for AAAA queries for %DOMAIN% and ipv4only.%DOMAIN% between resolv1 and resolv2

# Enable DNS64 on resolv2

module-config: "dns64 validator iterator"
dns64-prefix: 64:ff9b::/96
do-nat64: yes

# Testing

```
dig ipv4only.%DOMAIN% A
```
```
dig ipv4only.%DOMAIN% AAAA
```
```
dig ipv6only.%DOMAIN% A
```
```
dig ipv6only.%DOMAIN% AAAA
```
```
curl -4 http://%IPv4%
```
```
curl -6 http://[%IPv6%]
```
```
curl -6 http://[64:ff9b::%IPv4%]
```
```
curl http://ipv6only.%DOMAIN%
```
```
curl http://ipv4only.%DOMAIN%
```





---

layout: page
title: DNS64
parent: DNS
nav_order: 8
------------

# DNS64

In this lab we will configure **DNS64** and investigate how DNS64 and NAT64 allow an IPv6-only client to communicate with an IPv4-only server.

DNS64 and NAT64 solve two different parts of the problem:

* **DNS64** creates a synthetic IPv6 (AAAA) record from an IPv4 (A) record.
* **NAT64** translates the actual network traffic between IPv6 and IPv4.

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

You should find an A record, but no AAAA record.

Now query the IPv6-only host:

```bash
dig ipv6only.%DOMAIN% A
dig ipv6only.%DOMAIN% AAAA
```

This time you should find a AAAA record but no A record.

If a host has both IPv4 and IPv6 addresses, it can have both record types:

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

The IPv4 connection should work because the server has an IPv4 address.

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

An IPv6 client sends its packet towards an address containing the NAT64 prefix:

```text
IPv6 client
     |
     | IPv6
     | destination 64:ff9b::IPv4-address
     v
   NAT64
     |
     | IPv4
     | destination IPv4-address
     v
IPv4 server
```

The NAT64 translator extracts the IPv4 address from the IPv6 destination address and translates the packet to IPv4.

But there is still a problem.

How does an application know which NAT64 IPv6 address to use?

That is the job of DNS64.

# DNS64

DNS64 is implemented by a recursive DNS resolver.

When a client asks for an AAAA record and the destination has an IPv4 address but no IPv6 address, the DNS64 resolver can create a **synthetic AAAA record**.

For example, imagine that DNS contains:

```text
www.example.    IN A    192.0.2.1
```

but no AAAA record.

A normal recursive resolver receives:

```text
www.example. AAAA?
```

and returns no AAAA address.

A DNS64 resolver can instead construct:

```text
64:ff9b::192.0.2.1
```

and return this as a synthetic AAAA record.

The process looks like this:

```text
IPv6 client
     |
     | AAAA ipv4only.%DOMAIN%?
     v
 DNS64 resolver
     |
     | A ipv4only.%DOMAIN%?
     v
authoritative DNS
     |
     | IPv4 address
     v
 DNS64 resolver
     |
     | synthesize AAAA
     | 64:ff9b::IPv4-address
     v
IPv6 client
```

The client can then connect to this IPv6 address.

The packets are sent to the NAT64 translator, which translates them to IPv4.

# Establish a baseline

Before enabling DNS64, let's compare our two recursive resolvers.

`resolv1` has the IPv4 address:

```text
100.100.%GRP%.67
```

and `resolv2`:

```text
100.100.%GRP%.68
```

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

We are now going to change only `resolv1`.

# Enable DNS64 on resolv1

Log in to `resolv1`.

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

Your configuration should therefore contain something similar to:

```text
options {
    ...

    dns64 64:ff9b::/96 {
        clients { any; };
    };

    ...
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

We have enabled DNS64 on `resolv1`.

Do **not** enable it on `resolv2` yet.

We will use `resolv2` as a comparison.

# Compare resolv1 and resolv2

Ask both resolvers for the AAAA record of the IPv4-only host.

First `resolv1`:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA
```

Now `resolv2`:

```bash
dig @100.100.%GRP%.68 ipv4only.%DOMAIN% AAAA
```

Compare the ANSWER sections carefully.

Do both resolvers return the same answer?

`resolv1` should now return an address beginning with:

```text
64:ff9b::
```

`resolv2` should not return an AAAA address.

But both recursive resolvers are using the same authoritative DNS information.

So where did the additional AAAA record come from?

The answer is: **resolv1 created it**.

# Is the AAAA record really in DNS?

Let's verify that the authoritative server does not contain the AAAA record returned by `resolv1`.

Query your authoritative server directly:

```bash
dig @100.100.%GRP%.130 ipv4only.%DOMAIN% A
dig @100.100.%GRP%.130 ipv4only.%DOMAIN% AAAA
```

Now compare that with:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA
```

The authoritative server has the A record but no AAAA record.

Nevertheless, `resolv1` returns an AAAA record.

This is why we call the DNS64 answer a **synthetic AAAA record**.

DNS64 does not modify the authoritative DNS zone.

The synthetic record is created by the recursive resolver while answering the client's query.

# Find the IPv4 address

Look again at the synthesized AAAA record:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA
```

Now find the original IPv4 address:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% A
```

Compare the two addresses.

The DNS64 address consists of:

```text
+--------------------------+------------------+
|       64:ff9b::/96       |   IPv4 address   |
+--------------------------+------------------+
        96 bits                  32 bits
```

The final 32 bits represent the IPv4 destination.

For example:

```text
IPv4:

192.0.2.123
```

in hexadecimal is:

```text
c0 00 02 7b
```

and can therefore appear in an IPv6 address as:

```text
64:ff9b::c000:27b
```

Can you identify the IPv4 address embedded in the AAAA record returned for:

```text
ipv4only.%DOMAIN%
```

Does it match the A record?

# What happens when a real AAAA record exists?

DNS64 is needed when a host has an A record but no usable AAAA record.

Let's see what happens with an IPv6 host.

Run:

```bash
dig @100.100.%GRP%.67 ipv6only.%DOMAIN% AAAA
```

Compare this with:

```bash
dig @100.100.%GRP%.68 ipv6only.%DOMAIN% AAAA
```

Do the two resolvers return the same IPv6 address?

Is the address inside `64:ff9b::/96`?

A DNS64 resolver does not need to synthesize an address when the DNS response already contains a usable AAAA record.

The normal AAAA record is returned.

This gives us three important cases:

| DNS records | AAAA response from DNS64 |
| ----------- | ------------------------ |
| A only      | Synthetic AAAA           |
| AAAA only   | Original AAAA            |
| A and AAAA  | Original AAAA            |

# Test NAT64 directly

We have seen how DNS64 creates an IPv6 address.

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

Replace `%IPv4%` with the IPv4 address you found.

For example:

```text
http://[64:ff9b::192.0.2.1]
```

If this succeeds, DNS64 was not involved in this test.

You constructed the NAT64 destination address yourself.

This demonstrates an important distinction:

```text
DNS64
    creates an IPv6 destination address

NAT64
    translates packets sent to that address
```

# Test DNS64 and NAT64 together

Now let the application use DNS normally.

Make sure your client is using `resolv1` as its recursive resolver.

Run:

```bash
curl -6 http://ipv4only.%DOMAIN%
```

This time you supplied only a hostname.

What happened?

The complete process is:

```text
                         DNS
                          |
IPv6 client               |
     |                    |
     | AAAA query         |
     +------------------->|
                          |
                     DNS64 resolver
                          |
                          | A query
                          v
                   authoritative DNS
                          |
                          | A record
                          v
                     DNS64 resolver
                          |
                          | synthetic AAAA
                          v
IPv6 client <-------------+
     |
     |
     | IPv6 packet
     | destination 64:ff9b::IPv4-address
     |
     v
   NAT64
     |
     | IPv4 packet
     |
     v
IPv4-only web server
```

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

# Use resolv2 as a control

We deliberately left DNS64 disabled on `resolv2`.

Query:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA
```

and:

```bash
dig @100.100.%GRP%.68 ipv4only.%DOMAIN% AAAA
```

`resolv1` synthesizes an AAAA record.

`resolv2` does not.

This demonstrates that DNS64 is a function of the **recursive resolver**, not of the authoritative server.

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

# Enable DNS64 on resolv2

We kept `resolv2` unchanged so that we could compare a normal resolver with a DNS64 resolver.

Now configure DNS64 on `resolv2` as well.

Edit:

```bash
nano /etc/bind/named.conf.options
```

Add:

```text
dns64 64:ff9b::/96 {
    clients { any; };
};
```

Check the configuration:

```bash
named-checkconf
```

Reload BIND:

```bash
rndc reload
```

Now compare:

```bash
dig @100.100.%GRP%.67 ipv4only.%DOMAIN% AAAA
dig @100.100.%GRP%.68 ipv4only.%DOMAIN% AAAA
```

Both resolvers should now return synthetic AAAA records.

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

DNS64 operates in the DNS:

```text
ipv4only.%DOMAIN%
        |
        | A
        v
     IPv4 address
        |
        | DNS64
        v
64:ff9b::IPv4-address
```

DNS64 creates a synthetic AAAA record when a destination has an IPv4 address but no usable IPv6 address.

NAT64 operates on network packets:

```text
IPv6 client
     |
     | IPv6
     v
   NAT64
     |
     | IPv4
     v
IPv4 server
```

DNS64 does not translate packets, and NAT64 does not synthesize DNS records.

Together they allow an IPv6 application to use a normal hostname:

```text
curl -6 http://ipv4only.%DOMAIN%
```

even though the destination server itself only supports IPv4.
