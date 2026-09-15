---
layout: page
title: "Local Root"
parent: "DNS"
nav_order: 7
---

# Local Root

A recursive DNS resolver normally starts the resolution of a new name by contacting one of the DNS root servers.

For example, when a resolver needs to find an authoritative server for `example.com`, it first needs information from the root zone about `.com`.

The root zone is therefore an important part of every recursive DNS resolver.

In this lab we will configure a **local copy of the root zone** on the same server as the recursive resolver.

This mechanism is described in [RFC 8806](https://www.rfc-editor.org/rfc/rfc8806).

RFC 8806 is an Informational RFC. It describes a way to operate a copy of the root zone locally so that the resolver does not need to query a remote root server for root-zone information.

The local copy is still the real root zone. It is not a locally invented replacement for the root.

# Why serve the root locally?

A local root zone can provide several benefits.

It can reduce the number of queries that need to reach the root server system and can make root-zone lookups more resilient when remote root servers are unreachable.

It also means that queries for root-zone information do not have to cross the network to a remote root server.

There is an important operational cost:

> The local copy must be kept up to date.

A stale root zone can cause resolution problems.

RFC 8806 therefore requires the local copy to contain the complete root zone, including DNSSEC records, and requires the resolver to validate the data.

# Important security requirement

The local root service must not become an additional public root server.

RFC 8806 explicitly requires the local root service to answer only queries from the resolver on the same host. It must not provide authoritative root-zone service to other hosts.

This is why the configuration in this lab is deliberately different from an ordinary authoritative DNS server.

The local root data is there to support the recursive resolver.

It is not a service that other lab machines should use as a root server.

# Before changing anything

We already have two recursive resolvers:

```text
resolv1    100.100.%GRP%.67
resolv2    100.100.%GRP%.68
```

The client is configured to use them as DNS resolvers.

First verify that normal resolution is working.

From the `cli` machine run:

```bash
dig icann.org A
dig icann.org AAAA
dig org NS
dig com SOA
```

All queries should return successfully.

If they do not, fix the resolver configuration before continuing.

# Observe the current root hints

On your resolver, find out how the resolver currently knows about the root servers.

For BIND:

```bash
grep -R 'zone "."' /etc/bind/named.conf*
```

For Unbound:

```bash
grep -R 'root-hints' /etc/unbound/
```

BIND normally starts with root hints and uses them to locate the current root servers.

Unbound has built-in root hints and can also use an explicit `root-hints` file.

We will replace this normal starting point with a locally maintained copy of the root zone.

# Choose your implementation

You can implement the local root using either:

* [BIND](bind.md)
* [Unbound](unbound.md)

Use the instructions for the resolver software installed on your machine.

# Final questions

Make sure you can answer these questions:

1. What is contained in a root hints file?

2. What is contained in the root zone?

3. Why is a root hints file not the same thing as a local copy of the root zone?

4. Why does RFC 8806 require the complete root zone, including DNSSEC data?

5. Why must the local root service not answer requests from other hosts?

6. Does a local root mean that the resolver can operate completely without network access?

7. What happens if the local root zone becomes stale?

8. How does BIND maintain the local root zone?

9. How does Unbound maintain the local root zone?

10. Why is DNSSEC particularly important for a locally served root?
