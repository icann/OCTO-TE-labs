---

layout: page
title: "BIND"
parent: "Encrypted DNS"
nav_order: 1
------------

# Encrypted DNS with BIND

BIND 9 can provide encrypted DNS services directly.

In this lab we will configure:

* DNS over TLS (DoT)
* DNS over HTTPS (DoH)
* DDR

BIND currently supports DoT and DoH as server-side transports. Its listener configuration associates TLS and HTTP services with listening addresses.

BIND does not provide the same server-side DoQ functionality used in the Unbound part of this lab, so the DoQ exercise is performed using Unbound.

# Check the BIND version

Run:

```bash
named -v
```

Also check:

```bash
dig -v
```

The version of `dig` should support:

```text
+tls
+https
```

You can verify this with:

```bash
dig +help | grep -E 'tls|https'
```

# TLS certificate

BIND requires a TLS certificate and private key for encrypted DNS services.

We will use:

```text
/etc/bind/tls/resolver.key
/etc/bind/tls/resolver.pem
```

The certificate must contain:

```text
resolver.grp%GRP%.%DOMAIN%
```

in its Subject Alternative Name.

Check it:

```bash
openssl x509 \
    -in /etc/bind/tls/resolver.pem \
    -noout \
    -subject \
    -issuer \
    -dates \
    -ext subjectAltName
```

The private key must be readable by BIND but not by ordinary users.

Check:

```bash
ls -l /etc/bind/tls/resolver.key
```

# Create the TLS configuration

Edit:

```bash
nano /etc/bind/named.conf.local
```

Add:

```text
tls resolver-tls {
    key-file "/etc/bind/tls/resolver.key";
    cert-file "/etc/bind/tls/resolver.pem";
};
```

The `tls` statement defines the certificate and key used by BIND's encrypted DNS listeners. Current BIND documentation uses this mechanism for DoT and HTTPS listeners.

# Enable DNS over TLS

Edit:

```bash
nano /etc/bind/named.conf.options
```

Inside the `options` section, add:

```text
listen-on port 853 tls resolver-tls {
    100.100.%GRP%.67;
};

listen-on-v6 port 853 tls resolver-tls {
    %IPv6pfx%:%GRP%:64::67;
};
```

Use the actual IPv6 address of the resolver in the second block if the lab's address differs.

The listener now accepts:

```text
TCP/853
```

using TLS.

# Enable DNS over HTTPS

In the same `options` section, add:

```text
http default {
};
```

The `default` HTTP configuration causes BIND to use the standard:

```text
/dns-query
```

endpoint.

Now add HTTPS listeners:

```text
listen-on port 443 tls resolver-tls http default {
    100.100.%GRP%.67;
};

listen-on-v6 port 443 tls resolver-tls http default {
    %IPv6pfx%:%GRP%:64::67;
};
```

BIND requires TLS when an HTTP listener is configured, and the default HTTP endpoint is `/dns-query`.

# Check the configuration

Run:

```bash
named-checkconf
```

If the command reports an error, fix it before restarting BIND.

In particular, check:

* certificate filename
* private-key filename
* placement of the `tls` statement
* HTTP configuration
* listening addresses

# Restart BIND

Restart:

```bash
systemctl restart bind9
```

Check:

```bash
systemctl status bind9
```

If BIND does not start:

```bash
journalctl -u bind9 --no-pager -n 100
```

# Verify the listeners

Run:

```bash
ss -lntp | grep -E ':53|:443|:853'
```

You should see BIND listening on:

```text
53
853
443
```

The exact local addresses depend on your configuration.

# Test DoT

Run:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +tls \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

`dig +tls` uses DoT and defaults to port 853. `+tls-hostname` tells `dig` which hostname to validate in the TLS certificate.

The DNS answer should be identical to a normal query:

```bash
dig @100.100.%GRP%.67 www.%DOMAIN% A
```

# Test the TLS certificate

Run:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:853 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

Look for:

```text
subjectAltName
```

and:

```text
Verify return code
```

The certificate must authenticate the resolver name.

# Test DoH

Use:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +https \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

BIND `dig` uses `/dns-query` by default for DoH and HTTPS port 443 by default.

You can test the TLS endpoint separately:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:443 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

# Test DoH with a different URI

The default endpoint is:

```text
/dns-query
```

You can explicitly tell `dig` to use another endpoint:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +https=/dns-query \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

The endpoint is part of the DoH service description and is therefore also relevant to DDR.

# Configure DDR

DDR is implemented using SVCB records.

The special name is:

```text
_dns.resolver.arpa.
```

Create local DNS data for this name.

The simplest approach for this lab is to create a local zone:

```text
zone "resolver.arpa" {
    type primary;
    file "/var/lib/bind/zones/db.resolver.arpa";
};
```

Add this to:

```text
/etc/bind/named.conf.local
```

Create:

```bash
nano /var/lib/bind/zones/db.resolver.arpa
```

Use:

```text
$TTL 300
@       IN SOA soa.grp%GRP%.%DOMAIN%. dnsadmin.%DOMAIN%. (
                1
                300
                300
                300
                300
)

        IN NS soa.grp%GRP%.%DOMAIN%.

_dns    IN SVCB 1 resolver.grp%GRP%.%DOMAIN%. alpn=h2 dohpath=/dns-query{?dns}
_dns    IN SVCB 2 resolver.grp%GRP%.%DOMAIN%. alpn=dot port=853
```

These records advertise:

```text
Priority 1:
    DoH

Priority 2:
    DoT
```

RFC 9462 defines SVCB records using these service parameters for DDR discovery.

Do not add a DoQ record to the BIND configuration.

BIND is not providing DoQ in this lab.

# Make resolver.arpa local

Because `_dns.resolver.arpa.` is a DDR discovery name, queries for it must not accidentally be answered by unrelated upstream data.

RFC 9462 describes special local handling for `resolver.arpa`.

The local zone provides that behavior in the lab.

# Check the DDR zone

Run:

```bash
named-checkzone \
    resolver.arpa \
    /var/lib/bind/zones/db.resolver.arpa
```

Then:

```bash
named-checkconf
```

Reload:

```bash
rndc reload
```

# Test DDR

Query the SVCB record:

```bash
dig @100.100.%GRP%.67 \
    _dns.resolver.arpa. SVCB
```

You should receive the DoH and DoT records.

Check the resolver address:

```bash
dig resolver.grp%GRP%.%DOMAIN% A
dig resolver.grp%GRP%.%DOMAIN% AAAA
```

Now the client has enough information to discover the encrypted DNS services.

# Test the complete path

The intended sequence is:

```text
Client
  |
  | knows 100.100.%GRP%.67
  v
_dns.resolver.arpa. SVCB
  |
  | discovers
  v
resolver.grp%GRP%.%DOMAIN%
  |
  +---- DoH :443
  |
  +---- DoT :853
```

The client can now choose an encrypted DNS transport.

# Observe the difference

Compare:

```bash
dig @100.100.%GRP%.67 www.%DOMAIN% A
```

with:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +tls \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

and:

```bash
dig \
    @resolver.grp%GRP%.%DOMAIN% \
    +https \
    +tls-hostname=resolver.grp%GRP%.%DOMAIN% \
    www.%DOMAIN% A
```

The DNS answer should be the same.

The transport is different.

# Troubleshooting

Check BIND:

```bash
named-checkconf
systemctl status bind9
journalctl -u bind9 --no-pager -n 100
```

Check listeners:

```bash
ss -lntp | grep -E ':443|:853'
```

Check DoT:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:853 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

Check DoH:

```bash
openssl s_client \
    -connect resolver.grp%GRP%.%DOMAIN%:443 \
    -servername resolver.grp%GRP%.%DOMAIN%
```

Check DDR:

```bash
dig @100.100.%GRP%.67 \
    _dns.resolver.arpa. SVCB
```

# What you have learned

BIND can provide encrypted DNS without introducing another DNS server.

The configuration consists of:

```text
TLS certificate
       |
       +-- DoT listener TCP/853
       |
       +-- DoH listener TCP/443
```

and a DNS SVCB record that advertises the encrypted resolver:

```text
_dns.resolver.arpa.
```

DDR therefore connects the initial resolver configuration with the encrypted DNS services.
