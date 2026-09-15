---
layout: page
title: Encrypted DNS
parent: DNS
nav_order: 9
------------

# Encrypted DNS

Traditional DNS uses UDP or TCP port 53.

The DNS messages are not encrypted.

Anybody on the path can observe the query/response content.

Encrypted DNS provides confidentiality for the connection between the client and the resolver.

In this lab we will configure the recursive resolver itself to provide:

* DNS over TLS (DoT)
* DNS over HTTPS (DoH)
* DNS over QUIC (DoQ), where supported
* Discovery of Designated Resolvers (DDR)

We will configure both **BIND** and **Unbound**.

# The three encrypted DNS transports

## DNS over TLS (DoT)

DoT sends DNS messages over a TLS connection.

The standard service port is: TCP/853

The TLS certificate authenticates the resolver.

## DNS over HTTPS (DoH)

DoH carries DNS messages inside HTTPS.

The standard service port is: TCP/443

A client therefore communicates with the DNS resolver using HTTPS.

## DNS over QUIC (DoQ)

DoQ carries DNS messages over QUIC.

The standard service port is: TPC/853

QUIC provides encrypted transport using TLS 1.3.

BIND currently provides DoT and DoH server support. 

Unbound supports Dot, DoH and downstream DoQ. 


# Resolver names and certificates

Encrypted DNS needs a resolver name that clients can validate.

For this lab use:

```text
resolver.grp%GRP%.%DOMAIN%.
```

The resolver name must have A and AAAA records pointing to the resolver.

Check:

```bash
dig resolver.grp%GRP%.%DOMAIN% A
dig resolver.grp%GRP%.%DOMAIN% AAAA
```

The TLS certificate installed on the resolver must contain:

```text
resolver.grp%GRP%.%DOMAIN%
```

in its Subject Alternative Name.

Do not work around certificate validation by telling the client to ignore certificate errors.

One of the important purposes of encrypted DNS is authentication as well as confidentiality.

# Test the certificate

Before configuring encrypted DNS, inspect the certificate:

```bash
openssl x509 -in /path/to/certificate.pem -noout -subject -issuer -dates -ext subjectAltName
```

Verify that the resolver name appears in the Subject Alternative Name.

You can also inspect the TLS service directly after it has been enabled:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:853 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

Check:

```text
Certificate chain
Subject
Issuer
Subject Alternative Name
Verify return code
```

The certificate name must match the name used by the client.

# Baseline: normal DNS

Before configuring encrypted DNS, perform a normal DNS query:

```bash
dig @100.100.%GRP%.67 www.%DOMAIN% A
```

Then perform the same query using the other resolver:

```bash
dig @100.100.%GRP%.68 www.%DOMAIN% A
```

Normal DNS uses port 53.

We will use these queries as the baseline for the encrypted transports.

# Configure the resolver

Choose the implementation you are using:

* [BIND](bind.md)
* [Unbound](unbound.md)

Complete that configuration before continuing.

# Test DNS over TLS

After configuring DoT, test the service.

Use a DNS client that supports DoT.

BIND's `dig` supports DNS over TLS:

```bash
dig @resolver.grp%GRP%.%DOMAIN% \
    +tls \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

The default DoT port is 853.

A successful result demonstrates:

```text
DNS
  |
TLS
  |
TCP/853
```

The DNS answer should be the same answer you obtained using ordinary DNS.

The transport is different; the DNS data is not.

# Test DNS over HTTPS

BIND's `dig` supports DoH:

```bash
dig @resolver.grp%GRP%.%DOMAIN% \
    +https \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

The default endpoint is:

```text
/dns-query
```

and the default HTTPS port is 443.

The protocol stack is now:

```text
DNS
  |
HTTP/2
  |
TLS
  |
TCP/443
```

You can also verify the TLS connection independently:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:443 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

# Test DNS over QUIC

DoQ uses:

```text
UDP/853
```

The easiest way to test the service depends on the client software installed in the lab.

For Unbound, the source tree contains a `doqclient` test client when built with DoQ support. Current Unbound documentation describes this test client and the required arguments.

For example:

```bash
./doqclient \
    -s resolver.grp%GRP%.%DOMAIN% \
    -p 853 \
    www.%DOMAIN% A IN
```

DoQ is only available where the resolver has been built with the required QUIC support.

If the BIND resolver is being used for this exercise, document that DoQ is not available from BIND and perform the DoQ exercise against Unbound.

# What is DDR?

You now know how to configure an encrypted DNS service.

But there is another problem.

How does a client discover that an encrypted resolver exists?

Suppose the network configuration tells a client only:

```text
DNS resolver:
100.100.%GRP%.67
```

The client knows the IP address.

It does not know:

```text
resolver name
TLS certificate name
DoH URI
DoT port
DoQ port
```

DDR solves this discovery problem.

DDR stands for:

**Discovery of Designated Resolvers**

RFC 9462 defines DDR using SVCB records. A client initially configured with an ordinary DNS resolver can query:

```text
_dns.resolver.arpa.
```

for SVCB records.

The response can advertise DoT, DoH and DoQ services.

For example:

```dns
_dns.resolver.arpa.  IN SVCB  1 resolver.grp%GRP%.%DOMAIN%. (
        alpn=dot
        port=853
)
```

and:

```dns
_dns.resolver.arpa.  IN SVCB  2 resolver.grp%GRP%.%DOMAIN%. (
        alpn=h2
        dohpath=/dns-query{?dns}
)
```

and, when DoQ is available:

```dns
_dns.resolver.arpa.  IN SVCB  3 resolver.grp%GRP%.%DOMAIN%. (
        alpn=doq
        port=853
)
```

RFC 9462 defines these SVCB records and the `alpn`, `port`, and `dohpath` parameters used by encrypted-DNS clients.

# Important: DDR is not a transport

DDR does not encrypt the DNS traffic.

Instead:

```text
DoT
    encrypts DNS using TLS

DoH
    encrypts DNS using HTTPS/TLS

DoQ
    encrypts DNS using QUIC/TLS

DDR
    tells the client which encrypted resolver to use
```

That distinction is important.

# Query the DDR record

From the client:

```bash
dig @100.100.%GRP%.67 \
    _dns.resolver.arpa. SVCB
```

You should receive the SVCB records advertising the encrypted DNS services.

Inspect the answer carefully.

Identify:

```text
TargetName
ALPN
Port
DoH path
```

Then verify the address of the advertised resolver:

```bash
dig resolver.grp%GRP%.%DOMAIN% A
dig resolver.grp%GRP%.%DOMAIN% AAAA
```

# DDR and certificate validation

DDR is not simply a DNS redirect.

A client must be able to verify that the advertised resolver really is designated by the resolver it was originally configured to use.

This is why the resolver's certificate and the relationship between the original resolver and designated resolver matter.

RFC 9462 defines verification rules for discovered resolvers and specifically discusses certificate validation and the original resolver IP address.

For this lab, keep all encrypted services on the same resolver.

This makes the relationship easy to understand:

```text
100.100.%GRP%.67
       |
       | DDR
       v
resolver.grp%GRP%.%DOMAIN%
       |
       +-- DoT
       +-- DoH
       +-- DoQ
```

# resolver.arpa is special

DDR uses:

```text
resolver.arpa.
```

as a special-use domain.

The query:

```text
_dns.resolver.arpa. SVCB
```

is therefore not an ordinary query that should simply be forwarded to an upstream recursive resolver.

RFC 9462 specifies special handling for `resolver.arpa` for resolvers implementing DDR.

This is particularly important if the resolver is configured as a forwarder.

# Compare the transports

Perform the same DNS query using all available transports.

For example:

```text
Normal DNS:
    UDP/53

DoT:
    TCP/853

DoH:
    TCP/443

DoQ:
    UDP/853
```

Use:

```text
www.%DOMAIN% A
```

for all tests.

The answer should be the same.

Only the transport used to carry the DNS message changes.

# Observe the network traffic

Run a packet capture on the client:

```bash
tcpdump -n \
    'port 53 or port 853 or port 443'
```

Perform one query over normal DNS.

Observe:

```text
UDP/53
```

Perform the same query using DoT.

Observe:

```text
TCP/853
```

The DNS payload should no longer be readable as a normal DNS packet.

Perform the query using DoH.

Observe:

```text
TCP/443
```

The DNS query is inside HTTPS/TLS.

For DoQ:

```text
UDP/853
```

The DNS message is carried inside QUIC.

This is an excellent way to demonstrate the difference between the four mechanisms.

# Check that ordinary DNS still works

Encrypted DNS does not necessarily replace ordinary DNS.

Your resolver should still answer normal DNS queries unless you have deliberately configured it otherwise.

Test:

```bash
dig @100.100.%GRP%.67 www.%DOMAIN% A
```

and:

```bash
dig @100.100.%GRP%.68 www.%DOMAIN% A
```

The encrypted services provide additional ways for clients to reach the same resolver.

# Final questions

1. What does DNS over TLS encrypt?

2. What is the standard DoT port?

3. What protocol carries DNS over HTTPS?

4. What is the standard DoH port?

5. What transport does DoQ use?

6. What is the standard DoQ port?

7. Does DDR encrypt DNS traffic?

8. What is the purpose of `_dns.resolver.arpa.`?

9. What information does an SVCB record provide to an encrypted-DNS client?

10. Why does the resolver need a certificate?

11. Why must the certificate name match the resolver name used by the client?

12. What is the difference between an encrypted DNS transport and DDR?

13. Why can two resolvers return identical DNS answers while using different transports?

14. Why is it useful to run the same DNS query over UDP/53, TCP/853, HTTPS/443 and QUIC/853?

15. Which encrypted DNS services are available in BIND?

16. Which encrypted DNS services are available in Unbound?

# Summary

Encrypted DNS protects DNS traffic between the client and recursive resolver.

The three principal transports are:

```text
DoT    DNS over TLS
DoH    DNS over HTTPS
DoQ    DNS over QUIC
```

DDR is different:

```text
DDR    discovery of encrypted resolvers
```

A typical deployment can therefore look like:

```text
                    Client
                       |
             initially knows only
                       |
                100.100.%GRP%.67
                       |
                       | DDR
                       v
       resolver.grp%GRP%.%DOMAIN%
                       |
          +------------+------------+
          |            |            |
         DoT          DoH          DoQ
       TCP/853      TCP/443      UDP/853
```

BIND provides DoT and DoH server functionality.

Unbound provides DoT and DoH and, when built with the required libraries, downstream DoQ as well.

DDR uses SVCB records at `_dns.resolver.arpa.` to advertise designated encrypted resolvers and their supported protocols.
