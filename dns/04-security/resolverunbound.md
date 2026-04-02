---
layout: page
title: "Resolver Unbound"
parent: "Security"
nav_order: 6
---

# Resolver Security Unbound

Open resolvers have been a security challenge on the internet for
decades. Please do not add to it, secure your resolver, let it only 
be used by your own network.

## Current Situation

Let's start with trying out our current security.
Your own resolvers resolv1 and resolv2 have ip addresses 100.100.%GRP%.67
and 100.100.%GRP%.68.

Try to access the resolver of some of your peer groups the ip-addresses
follow the same scheme as yours does.
```
dig @100.100.?.67 icann.org +noall +nocomments +answer
dig @100.100.?.68 icann.org +noall +nocomments +answer
```
- Do you get an answer?
- Do the others get answers from your resolvers?

## Access Control

Please change your `unbound.conf` and add the following section
```
        access-control: 100.100.%GRP%.0/24 allow
        access-control: 127.0.0.0/8 allow
        access-control: %IPv6pfx%:%GRP%::/48 allow
        access-control: ::1/128 allow
```
Check your configuration and reload the server
```
unbound-checkconf
sudo unbound-control reload
```

Run the same tests as above. Can you still access your peers resolvers?
Can they access yours? 

## Rate Limit

Clients can get out of control or deliberately attack you.
It is a good idea to limit clients to not use more than their
fair share of the resolvers resources.

In `unbound.conf` please add the following statements to the server section
```
    ratelimit: 1000;           # Max queries/sec per /64 IPv6 or /24 IPv4 (default 1000)
    ratelimit-size: 4m;        # Hash table size (default 4m)
    ratelimit-slabs: 2;        # Threads (power of 2; default 2)
    ratelimit-factor: 10;      # Multiplier for slip (default 10)
    ip-ratelimit: 100;         # Per-IP limit (default off)
    ip-ratelimit-size: 4m;
    ip-ratelimit-slabs: 2;
    ip-ratelimit-factor: 10;
    wait-limit: 700;           # Max waiting recursions per netblock (default 700)
    wait-limit-cookie: 10000;  # Higher for cookie clients (default 10000)
```

