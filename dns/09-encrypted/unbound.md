---
layout: page
title: "Unbound"
parent: "Encrypted DNS"
nav_order: 2
------------

# Encrypted DNS with Unbound

Unbound can provide encrypted DNS services directly.

In this lab we will configure:

* DNS over TLS (DoT)
* DNS over HTTPS (DoH)
* DNS over QUIC (DoQ)
* DDR

Current Unbound documentation provides downstream DoT, DoH and DoQ support. DoQ was introduced in Unbound 1.22.0 and requires QUIC support to be compiled into Unbound. Current Unbound releases use `libngtcp2` for DoQ, and OpenSSL 3.5 or newer can provide the required QUIC cryptography.

# Check the Unbound version

Run:

```bash
unbound -V
```

Look at the configure options and linked libraries.

For DoH, Unbound needs `libnghttp2`.

For DoQ, check that QUIC support is present.

If your installed Unbound does not have DoQ support, do not modify the system installation during the lab unless instructed to do so.

The DoQ section is conditional on the installed Unbound build.

# TLS certificate

Unbound uses:

```text
tls-service-key
tls-service-pem
```

for encrypted downstream services.

Use:

```text
/etc/unbound/tls/resolver.key
/etc/unbound/tls/resolver.pem
```

The certificate must contain:

```text
resolver.grp%GRP%.%DOMAIN%
```

in its Subject Alternative Name.

Check:

```bash
openssl x509 \
    -in /etc/unbound/tls/resolver.pem \
    -noout \
    -subject \
    -issuer \
    -dates \
    -ext subjectAltName
```

# Enable the TLS service

Create:

```bash
nano /etc/unbound/unbound.conf.d/encrypted.conf
```

Add:

```text
server:
    tls-service-key: "/etc/unbound/tls/resolver.key"
    tls-service-pem: "/etc/unbound/tls/resolver.pem"
```

Unbound uses this certificate and key for its TLS-based downstream services. This includes DoT and DoH.

# Enable DNS over TLS

Add:

```text
server:
    interface: 100.100.%GRP%.67@853
    interface: %IPv6pfx%:%GRP%:64::67@853
    tls-port: 853
```

The resolver now listens for DNS-over-TLS connections on TCP/853.

# Enable DNS over HTTPS

Add:

```text
server:
    interface: 100.100.%GRP%.67@443
    interface: %IPv6pfx%:%GRP%:64::67@443
    https-port: 443
```

Unbound's DoH implementation uses HTTP/2 and requires `libnghttp2`.

The HTTPS service uses:

```text
/dns-query
```

as its DNS-over-HTTPS endpoint.

# Enable DNS over QUIC

First verify that your Unbound build contains DoQ support.

If it does, add:

```text
server:
    interface: 100.100.%GRP%.67@853
    interface: %IPv6pfx%:%GRP%:64::67@853
    quic-port: 853
```

The `quic-port` option provides DNS-over-QUIC on UDP for the interfaces configured with that port. Unbound documents 853 as the default QUIC port.

The same TLS key and certificate are used:

```text
tls-service-key
tls-service-pem
```

for DoQ.

Notice that TCP and UDP can both use port 853:

```text
TCP/853  -> DoT
UDP/853  -> DoQ
```

That is intentional.

# Check the configuration

Run:

```bash
unbound-checkconf
```

If the configuration is valid, restart:

```bash
systemctl restart unbound
```

Check:

```bash
systemctl status unbound
```

If it does not start:

```bash
journalctl -u unbound --no-pager -n 100
```

# Verify the listeners

Run:

```bash
ss -lntup | grep -E ':443|:853'
```

You should see:

```text
TCP/443
TCP/853
```

and, when DoQ is supported:

```text
UDP/853
```

# Test DNS over TLS

Use a client that supports DoT.

With BIND's `dig`:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +tls \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

Compare with ordinary DNS:

```bash
dig @100.100.%GRP%.67 www.%DOMAIN% A
```

The DNS answer should be the same.

# Test DNS over HTTPS

With a recent BIND `dig`:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +https \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

BIND's `dig` uses `/dns-query` and HTTPS port 443 by default.

# Test DNS over QUIC

Unbound provides a test DoQ client when built from source.

From the Unbound source directory:

```bash
make doqclient
```

Then:

```bash
./doqclient \
    -s resolver.grp%GRP%.%DOMAIN% \
    -p 853 \
    www.%DOMAIN% A IN
```

The current Unbound documentation describes this test client and its command-line syntax.

If your installed Unbound was not built with DoQ support, this command will not be available.

Do not treat that as a configuration error.

Verify the build first.

# Test DoQ using packet capture

Run:

```bash
tcpdump -n udp port 853
```

Then perform the DoQ query.

You should see UDP traffic to port 853.

Compare this with DoT:

```bash
tcpdump -n tcp port 853
```

DoT:

```text
TCP/853
```

DoQ:

```text
UDP/853
```

This is a useful demonstration that DoQ changes the transport rather than merely changing the encryption configuration of DoT.

# Configure DDR

DDR uses:

```text
_dns.resolver.arpa.
```

with SVCB records.

For the lab we will serve these records locally.

Create:

```bash
nano /etc/unbound/unbound.conf.d/ddr.conf
```

Add an authority zone for:

```text
resolver.arpa.
```

For example:

```text
server:
    local-zone: "resolver.arpa." static

    local-data: '_dns.resolver.arpa. 300 IN SVCB 1 resolver.grp%GRP%.%DOMAIN%. alpn=h2 dohpath=/dns-query{?dns}'
    local-data: '_dns.resolver.arpa. 300 IN SVCB 2 resolver.grp%GRP%.%DOMAIN%. alpn=dot port=853'
    local-data: '_dns.resolver.arpa. 300 IN SVCB 3 resolver.grp%GRP%.%DOMAIN%. alpn=doq port=853'
```

These records advertise:

```text
DoH
DoT
DoQ
```

RFC 9462 defines the use of SVCB records for this purpose.

# Why are these local records?

DDR uses a special-purpose namespace:

```text
resolver.arpa.
```

A recursive forwarder should not simply forward these discovery queries to an unrelated upstream resolver.

The purpose of the records is to describe the encrypted services designated by this resolver.

The SVCB records therefore belong to the local resolver configuration.

RFC 9462 specifically discusses special handling for `resolver.arpa`.

# Restrict resolver.arpa

The local zone should behave as a locally served zone.

For queries other than the DDR SVCB query:

```text
_dns.resolver.arpa. SVCB
```

the resolver should not accidentally expose unrelated upstream data.

RFC 9462 specifies NODATA behavior for other query types and names beneath `resolver.arpa.`.

# Check the configuration

Run:

```bash
unbound-checkconf
```

Then restart:

```bash
systemctl restart unbound
```

# Test DDR

Run:

```bash
dig @100.100.%GRP%.67 \
    _dns.resolver.arpa. SVCB
```

You should see the three advertised services:

```text
DoH
DoT
DoQ
```

Check the target:

```bash
dig resolver.grp%GRP%.%DOMAIN% A
dig resolver.grp%GRP%.%DOMAIN% AAAA
```

The client now has the information required to select an encrypted resolver.

# Compare the SVCB records

Look at the three responses carefully.

DoH:

```text
alpn=h2
dohpath=/dns-query{?dns}
```

DoT:

```text
alpn=dot
port=853
```

DoQ:

```text
alpn=doq
port=853
```

The records therefore tell the client both:

```text
which protocol
```

and, where required:

```text
where and how to connect
```

RFC 9462 specifies these SVCB service parameters for encrypted DNS discovery.

# Test the complete DDR path

The conceptual sequence is:

```text
Client
  |
  | configured with 100.100.%GRP%.67
  |
  | SVCB _dns.resolver.arpa.
  v
Unbound
  |
  | returns designated resolver
  v
resolver.grp%GRP%.%DOMAIN%
  |
  +---- DoH
  |
  +---- DoT
  |
  +---- DoQ
```

DDR therefore allows the encrypted service to be discovered without requiring the client to have been manually configured with the encrypted resolver hostname.

# Certificate validation

DDR depends on authentication.

Check the certificate:

```bash
openssl x509 \
    -in /etc/unbound/tls/resolver.pem \
    -noout \
    -ext subjectAltName
```

The certificate must contain:

```text
resolver.grp%GRP%.%DOMAIN%
```

A client should not blindly trust an arbitrary encrypted resolver merely because an SVCB record says to use it.

RFC 9462 includes certificate-based verification of discovered designated resolvers.

# DoT, DoH and DoQ comparison

Use the same DNS question:

```text
www.%DOMAIN% A
```

Compare:

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

All four should ultimately provide the same DNS answer.

The important difference is the transport used between client and resolver.

# Observe the network

Start a capture:

```bash
tcpdump -n \
    'port 53 or port 443 or port 853'
```

Perform:

```text
normal DNS
DoT
DoH
DoQ
```

Look at the resulting traffic.

You should be able to distinguish:

```text
UDP/53
TCP/853
TCP/443
UDP/853
```

The encrypted transports should not expose the DNS question as a normal DNS packet.

# Check statistics

Unbound exposes statistics for the different transports.

Run:

```bash
unbound-control stats_noreset | grep -E 'tls|https|quic'
```

The exact counters depend on the Unbound version and build.

For DoQ, current documentation identifies:

```text
num.query.quic
```

as the number of QUIC queries.

# Troubleshooting

Check the configuration:

```bash
unbound-checkconf
```

Check the service:

```bash
systemctl status unbound
```

Check listeners:

```bash
ss -lntup | grep -E ':443|:853'
```

Check the TLS service:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:853 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

Check DDR:

```bash
dig @100.100.%GRP%.67 \
    _dns.resolver.arpa. SVCB
```

Check DoQ support:

```bash
unbound -V
```

If DoQ is unavailable, verify that the installed Unbound was compiled with QUIC support.

Current Unbound DoQ documentation identifies `libngtcp2` and suitable OpenSSL support as build requirements.

# What you have learned

Unbound can provide:

```text
DoT
    TCP/853

DoH
    TCP/443

DoQ
    UDP/853
```

and can advertise these services using DDR SVCB records:

```text
_dns.resolver.arpa.
```

The important concepts are:

```text
Encrypted DNS transport
        +
TLS certificate authentication
        +
DDR service discovery
```

Together these allow a client to discover and use an authenticated encrypted DNS resolver without requiring the encrypted service to be manually configured in advance.
    