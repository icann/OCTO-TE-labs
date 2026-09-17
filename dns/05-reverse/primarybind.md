---
layout: page
title: "Primary Bind"
parent: "Reverse DNS"
nav_order: 1
---

# Configure the primary authoritative server (SOA)

To map your IP addresses to your domain names, we need to set up a reverse zone.

We are going to configure the hidden authoritative server for the reverse zone:

```
%GRP%.100.100.in-addr.arpa.
```

On your `soa` server create the zone file:

```
nano /var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa
```

Add:

```
$TTL    30
@       IN      SOA     soa.grp%GRP%.%DOMAIN%. dnsadmin.%DOMAIN%. (
                              1         ; Serial
                             30         ; Refresh
                             30         ; Retry
                             30         ; Expire
                             30 )       ; Negative Cache TTL
;

        IN      NS      ns1.grp%GRP%.%DOMAIN%.
        IN      NS      ns2.grp%GRP%.%DOMAIN%.

2       IN      PTR     cli.grp%GRP%.%DOMAIN%.
66      IN      PTR     soa.grp%GRP%.%DOMAIN%.
67      IN      PTR     resolv1.grp%GRP%.%DOMAIN%.
68      IN      PTR     resolv2.grp%GRP%.%DOMAIN%.
70      IN      PTR     rpki.grp%GRP%.%DOMAIN%.
130     IN      PTR     ns1.grp%GRP%.%DOMAIN%.
131     IN      PTR     ns2.grp%GRP%.%DOMAIN%.
```

Save and exit.

Run the following command to check for any errors in your zone:

```
named-checkzone %GRP%.100.100.in-addr.arpa /var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa
```

Next, edit:

```
nano /etc/bind/named.conf.local
```

and add:

```
zone "%GRP%.100.100.in-addr.arpa" {
    type primary;
    file "/var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa";
    allow-transfer { any; };
    also-notify {
        100.100.%GRP%.130;
        100.100.%GRP%.131;
        %IPv6pfx%:%GRP%:128::130;
        %IPv6pfx%:%GRP%:128::131;
    };
};
```

Save and exit.

Check the BIND configuration:

```
named-checkconf
```

Load the new bind configuration:

```
rndc reconfigure
```

Now test your reverse DNS:

```
dig @localhost -x 100.100.%GRP%.66
```

You can also make exactly the same query without using the `-x` option:

```
dig @localhost 66.%GRP%.100.100.in-addr.arpa. PTR
```

Did you get a DNS response with the PTR record in the answer section?
