---
layout: page
title: "Resolver Bind"
parent: "Security"
nav_order: 5
---

# Resolver Security Bind

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

Please change your `named.conf.options` and add the following section
```
acl "trusted" {
    localhost; 100.100.%GRP%.0/24; fd89:59e0:%GRP%::/48; 
};
```
And edit the to following line to look like this
```
    allow-query { trusted; };
    allow-recursion { trusted; };
```
Check your configuration and reload the server
```
named-confcheck
sudo rndc reload
```

Run the same tests as above. Can you still access your peers resolvers?
Can they access yours? 

## Rate Limit

Clients can get out of control or deliberately attack you.
It is a good idea to limit clients to not use more than their
fair share of the resolvers resources.

In `named.conf.options` please add the following section
```
rate-limit {
    responses-per-second 1;  
    exempt-clients { localhost; };
    //log-only yes;
    qps-scale 100; 
    window 60;  // seconds
    ipv4-prefix-length 24;
    ipv6-prefix-length 48;
    max-table-size 1000;
};
```

Now let's test the rate limit.

dnsperf 
