---
layout: page
title: "Resolver Resilience"
parent: "Validation"
nav_order: 3
---

# Resolver Resilience

Resolvers are central to any larger server installation. Only with proper resolving can servers communicate with each other. DNSSEC introduces some new dependencies to the process of resolving.

## Time

DNSSEC signatures have an inception and and expiration time. So I resolver has to check if 
the current time falls between inception and expiration. For this the resolver has to know
what the current time is.

When a resolver cold boots knowing the time can be a problem. This is usually solved by 
using NTP. But what is the first thing an NTP client will do? A DNS request.

So we are caught in a catch22 situation. DNS depends on NTP and NTP depends on DNS.

This can be solved by using IP addresses for the NTP configuration.

## Island Operations

There will be time for every datacenter when an unlucky building crew managed to cut all outside cable connections. Connection to the internet is lost, Island Operations start.

For internal servers to still be able to reach other internal resources or for sys-admins to be able to reach internal resources DNS can not stop working.

To make internal names run in Island Operations an authoritative name server for your internal zone(s) has to be present in the datacenter and be configured in your resolver.

In this lab installation we use .internal and we have configured .internal in resolv1/2. 

> [!NOTICE] .internal has been signified as a Top Level Domain that will never be deployed 
> in the root zone and can savely be used by anyone for internal purposes.

But DNSSEC introduces more dependencies. To be able to validate, a resolver needs the chain
of trust from a configured trust-anchor down to the data.

Usually we think of this as **the** trust-anchor. The one trust-anchor for the root.
But resolvers actually support using several trust anchors and thay can be for abitrary 
zones.

Let's start with looking at how we did setup .internal. Please run these commands on your cli machine.
```
dig internal SOA +dnssec
```
As you can see, the domain is DNSSEC signed. Did you get the AD flag? Of course not, .internal is not in the root, so there can't be a chain of trust. Let's change that.

Let's retrieve the public key
```
dig internal dnskey +noall +nocomments +answer
```
The output should look something like
```
internal. 300 IN DNSKEY	257 3 13 GeVBG8...50OyC6g==
```
Now we update our resolver configuration.

### Bind
In bind we edit the `named.conf.options`
```
sudo nano /etc/bind/named.conf.options
```
Please add the following section add the end. Please replace the data with the data you retrieved for above.
```
trust-anchors {
    "internal." static-key 257 3 13 "GeVBG8...50OyC6g==";
};
```
Let's restart the server
```
rndc reconfig
```
Run the query from the beginning again.
```
dig internal SOA +dnssec
```
Now you should see the AD flag.

### Unbound

In the server section of `unbound.conf`
```
sudo nano /etc/unbound/unbound.conf
```
please add the following statement (again use the values you retrieved from internal earlier)
```
        trust-anchor:
                "internal. 300 IN DNSKEY 257 3 13 GeVBG8...50OyC6g=="
```

Now unbound should show the AD flag too.
