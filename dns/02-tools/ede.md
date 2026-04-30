---
layout: page
title: "EDE"
parent: "DNS Tools"
nav_order: 3
---

# Extended DNS Errors (EDE)

A few years back the Internet Engineering Task Force (IETF) standardized
a new way for DNS system to be more specific about errors.

Up to that point many errors were mainly communicated through the status code "SERVFAIL".
But it could mean anything, from non-reachable name servers to DNSSEC validation errors.
So [RFC 8914 Extended DNS Errors](https://www.rfc-editor.org/rfc/rfc8914) fixes this.

The list of all possible EDE values can be found at the [IANA Registry for Extended DNS Error Codes](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#extended-dns-error-codes)

> [!IMPORTANT]
> Not all implementations use these error codes in the same way.

Test the following commands and see which error messages you get.
```
dig @9.9.9.9 dnssec-failed.org
dig @1.1.1.1 dnssec-failed.org
```
On the web site (https://extended-dns-errors.com/) a list of domains with a lot of different
errors are listed. Try accessing them through different resolvers. Which error messages
do you get back?

You can use public resolvers like
1. Quad9 9.9.9.9
1. Google 8.8.8.8
1. Cloudflare 1.1.1.1
1. DNS4EU 86.54.11.11
1. DNS.WATCH 84.200.69.80
 