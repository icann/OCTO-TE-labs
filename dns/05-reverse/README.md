---
layout: page
title: "Reverse DNS"
parent: "DNS"
nav_order: 5
---

# Reverse DNS

Everybody knows that the DNS can be used to translate names to ip numbers. 
But actually, the DNS can also translate IP numbers to names. This is done 
with the Reverse DNS.

# Configure the primary authoritative server (SOA)

To map your IP address to your domain name, we’ll need to setup a reverse zone. 
We are going to configure a hidden authoritative server for your reverse zone 
and create the authoritative zone.

Your 
```
nano /var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa
```
2.0.0.0  ::    8.2.1.0 RevGrp RevPfx
%IPv6pfx%:%GRP%:128::2
```◊
$TTL    30
@       IN      SOA     %DOMAIN%. te-labs.icann.org. (                                            
(                                            
                              1         ; Serial
                             30         ; Refresh
                             30         ; Retry
                             30         ; Expire
                             30 )       ; Negative Cache TTL
;

        IN      NS      ns1.grp%GRP%.%DOMAIN%. ; your name server
        IN      NS      ns2.grp%GRP%.%DOMAIN%. ; your name server
66		IN		PTR		soa.grp%GRP%.%DOMAIN%.
67		IN		PTR		resolv1.grp%GRP%.%DOMAIN%.
68		IN		PTR		resolv2.grp%GRP%.%DOMAIN%.
130		IN		PTR		ns1.grp%GRP%.%DOMAIN%.
131		IN		PTR		ns2.grp%GRP%.%DOMAIN%.



cli         A           100.100.%GRP%.2
cli         AAAA        %IPv6pfx%:%GRP%:128::2
soa         A           100.100.%GRP%.66
soa         AAAA        %IPv6pfx%:%GRP%:128::66
resolv1     A           100.100.%GRP%.67
resolv1     AAAA        %IPv6pfx%:%GRP%:128::67
resolv2     A           100.100.%GRP%.68
resolv2     AAAA        %IPv6pfx%:%GRP%:128::68
rpki        A           100.100.%GRP%.70
rpki        AAAA        %IPv6pfx%:%GRP%:128::70
ns1         A           100.100.%GRP%.130
ns1         AAAA        %IPv6pfx%:%GRP%:128::130
ns2         A           100.100.%GRP%.131
ns2         AAAA        %IPv6pfx%:%GRP%:128::131
```

Save and exit.

Run the following command to check for any errors in your setup:

```
named-checkzone %GRP%.100.100.in-addr.arpa /var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa
```

Next, edit the /etc/bind/named.conf.local file and add the following lines:

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

Run the following command to check for any errors in your setup:

```
named-checkconf
```

Restart Bind and test your reverse DNS using dig

```
dig @localhost -x 100.100.%GRP%.66 
```

Or

```
dig @localhost 66.%GRP%.100.100.in-addr.arpa. PTR
```

Did you get a DNS response with the PTR record in the answer section?

## Configure the secondary authoritative servers (ns1 and ns2) 

These servers are the ones that expose our (reverse) zone publicly (so they will be open-to-all servers). You should now know how to configure the secondary NS. If you forgot, go back to the lab where you created the forward zone for your grp%GRP% and follow the instructions to do this new configuration for your reverse zone.

Once you are done with configuration, test your reverse zone propagation.

## Test your zone configuration and propagation.
Use *dig* tool to verify your zone configuration and propagation, then do the same for one or two other groups in the class and share comments. From your client, run the following dig queries. All should return answer otherwise you should review your configurations before continiuing:

1. `dig @100.100.%GRP%.66  -x 100.100.%GRP%.66`
1. `dig @100.100.%GRP%.130 -x 100.100.%GRP%.66`
1. `dig @100.100.%GRP%.131 -x 100.100.%GRP%.66`
1. `dig @100.100.%GRP%.130 -x 100.100.%GRP%.67`
1. `dig @100.100.%GRP%.130 -x 100.100.%GRP%.68`




1. reverse delegation at octet boundary
2. keep forward and reverse in same zone
3. reverse delegation at non octet boundary












---

layout: page
title: Reverse DNS
parent: DNS
nav_order: 5
------------

# Reverse DNS

Everybody knows that the DNS can be used to translate names to IP addresses. But DNS can also translate IP addresses to names. This is called **Reverse DNS**.

For IPv4, reverse DNS uses the special domain `in-addr.arpa`.

For example, to find the name associated with:

```
100.100.%GRP%.66
```

a resolver asks for a PTR record for:

```
66.%GRP%.100.100.in-addr.arpa.
```

Notice that the octets of the IP address appear in reverse order.

The `dig` command can construct this name automatically:

```
dig -x 100.100.%GRP%.66
```

In this lab we will first configure a normal reverse DNS zone. We will then look at what happens when we want to delegate reverse DNS for a network that does **not** end at a DNS label boundary.

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

66      IN      PTR     soa.grp%GRP%.%DOMAIN%.
67      IN      PTR     resolv1.grp%GRP%.%DOMAIN%.
68      IN      PTR     resolv2.grp%GRP%.%DOMAIN%.
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

Restart BIND:

```
systemctl restart bind9
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

# Configure the secondary authoritative servers (ns1 and ns2)

These servers expose our reverse zone publicly.

You should now know how to configure secondary authoritative servers. If you forgot, go back to the lab where you created the forward zone for your `grp%GRP%` domain and follow the instructions to configure `ns1` and `ns2` as secondary servers for:

```
%GRP%.100.100.in-addr.arpa.
```

Once you are done, test that the reverse zone has propagated to both authoritative servers.

# Test your zone configuration and propagation

Use `dig` to verify your zone configuration and propagation.

From your client, run the following queries:

```
dig @100.100.%GRP%.66  -x 100.100.%GRP%.66
dig @100.100.%GRP%.130 -x 100.100.%GRP%.66
dig @100.100.%GRP%.131 -x 100.100.%GRP%.66
dig @100.100.%GRP%.130 -x 100.100.%GRP%.67
dig @100.100.%GRP%.130 -x 100.100.%GRP%.68
```

All queries should return an answer.

If they don't, review your configuration before continuing.

Try the same queries for one or two other groups in the class and compare the results.

# Classless reverse DNS delegation

So far, our entire network:

```
100.100.%GRP%.0/24
```

has been represented by the reverse DNS zone:

```
%GRP%.100.100.in-addr.arpa.
```

This works nicely because `/24` happens to correspond to a boundary between labels in the reverse DNS name.

But our network is actually divided into smaller networks.

One of these is the internal server network:

```
100.100.%GRP%.64/26
```

It contains the addresses:

```
100.100.%GRP%.64 - 100.100.%GRP%.127
```

Suppose this network is operated by another organization and we want to delegate responsibility for its reverse DNS to that organization.

How can we do that?

## DNS delegation happens at label boundaries

Let's look again at the reverse name for one of our servers:

```
100.100.%GRP%.67
```

Its reverse DNS name is:

```
67.%GRP%.100.100.in-addr.arpa.
```

DNS delegation can only happen between DNS labels.

For example, we can delegate:

```
%GRP%.100.100.in-addr.arpa.
```

because `%GRP%` is a complete DNS label.

But there is no DNS label representing:

```
100.100.%GRP%.64/26
```

The subnet boundary is inside the last octet.

We therefore cannot simply create a delegation for the `/26` in the normal way.

## RFC 2317

RFC 2317 describes a method called **Classless IN-ADDR.ARPA delegation**.

The basic idea is to create an additional DNS zone representing the subnet and use CNAME records in the parent reverse zone to redirect queries into it.

For our `/26`, we will use the zone name:

```
64-127.%GRP%.100.100.in-addr.arpa.
```

The name `64-127` is just a DNS label.

There is nothing special about the `-` character and DNS does not interpret `64-127` as an IP address range.

It is simply a name that the parent and child use for the classless reverse delegation.

Our setup will look like this:

```
%GRP%.100.100.in-addr.arpa.
|
|  CNAME
|
+-- 66  ---> 66.64-127.%GRP%.100.100.in-addr.arpa.
+-- 67  ---> 67.64-127.%GRP%.100.100.in-addr.arpa.
+-- 68  ---> 68.64-127.%GRP%.100.100.in-addr.arpa.
|
+-- 130 PTR ns1.grp%GRP%.%DOMAIN%.
+-- 131 PTR ns2.grp%GRP%.%DOMAIN%.
|
+-- 64-127
       |
       +-- delegated to ns1.grp%GRP%.%DOMAIN%.
       +-- delegated to ns2.grp%GRP%.%DOMAIN%.
```

The new delegated zone contains the actual PTR records:

```
64-127.%GRP%.100.100.in-addr.arpa.
|
+-- 66 PTR soa.grp%GRP%.%DOMAIN%.
+-- 67 PTR resolv1.grp%GRP%.%DOMAIN%.
+-- 68 PTR resolv2.grp%GRP%.%DOMAIN%.
```

Let's configure it.

# Create the classless reverse zone

We will first create the new reverse zone on the `soa` server.

Create:

```
nano /var/lib/bind/zones/db.64-127.%GRP%.100.100.in-addr.arpa
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

66      IN      PTR     soa.grp%GRP%.%DOMAIN%.
67      IN      PTR     resolv1.grp%GRP%.%DOMAIN%.
68      IN      PTR     resolv2.grp%GRP%.%DOMAIN%.
```

Save and exit.

Check the zone:

```
named-checkzone 64-127.%GRP%.100.100.in-addr.arpa /var/lib/bind/zones/db.64-127.%GRP%.100.100.in-addr.arpa
```

Now add the new zone to:

```
/etc/bind/named.conf.local
```

Add:

```
zone "64-127.%GRP%.100.100.in-addr.arpa" {
    type primary;
    file "/var/lib/bind/zones/db.64-127.%GRP%.100.100.in-addr.arpa";
    allow-transfer { any; };
    also-notify {
        100.100.%GRP%.130;
        100.100.%GRP%.131;
        %IPv6pfx%:%GRP%:128::130;
        %IPv6pfx%:%GRP%:128::131;
    };
};
```

Check the configuration:

```
named-checkconf
```

Restart BIND:

```
systemctl restart bind9
```

Test the new zone directly:

```
dig @localhost 67.64-127.%GRP%.100.100.in-addr.arpa. PTR
```

You should receive:

```
67.64-127.%GRP%.100.100.in-addr.arpa. IN PTR resolv1.grp%GRP%.%DOMAIN%.
```

Notice that this is **not yet** what a normal reverse lookup asks for.

A normal reverse lookup still asks for:

```
67.%GRP%.100.100.in-addr.arpa.
```

We need to connect these two names.

# Delegate the classless reverse zone

Open the original reverse zone:

```
nano /var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa
```

First, increment the SOA serial number.

Then add a delegation for the new zone:

```
64-127  IN      NS      ns1.grp%GRP%.%DOMAIN%.
64-127  IN      NS      ns2.grp%GRP%.%DOMAIN%.
```

This delegates:

```
64-127.%GRP%.100.100.in-addr.arpa.
```

to your authoritative name servers.

Now we need to redirect the individual reverse names into the delegated zone.

Remove these PTR records from the original zone:

```
66      IN      PTR     soa.grp%GRP%.%DOMAIN%.
67      IN      PTR     resolv1.grp%GRP%.%DOMAIN%.
68      IN      PTR     resolv2.grp%GRP%.%DOMAIN%.
```

Replace them with:

```
66      IN      CNAME   66.64-127.%GRP%.100.100.in-addr.arpa.
67      IN      CNAME   67.64-127.%GRP%.100.100.in-addr.arpa.
68      IN      CNAME   68.64-127.%GRP%.100.100.in-addr.arpa.
```

Do **not** change the PTR records for `130` and `131`.

They are outside the `100.100.%GRP%.64/26` network and remain part of the original `/24` reverse zone.

Your original reverse zone should now contain:

```
$TTL    30
@       IN      SOA     soa.grp%GRP%.%DOMAIN%. dnsadmin.%DOMAIN%. (
                              2         ; Serial
                             30         ; Refresh
                             30         ; Retry
                             30         ; Expire
                             30 )       ; Negative Cache TTL
;

        IN      NS      ns1.grp%GRP%.%DOMAIN%.
        IN      NS      ns2.grp%GRP%.%DOMAIN%.

64-127  IN      NS      ns1.grp%GRP%.%DOMAIN%.
64-127  IN      NS      ns2.grp%GRP%.%DOMAIN%.

66      IN      CNAME   66.64-127.%GRP%.100.100.in-addr.arpa.
67      IN      CNAME   67.64-127.%GRP%.100.100.in-addr.arpa.
68      IN      CNAME   68.64-127.%GRP%.100.100.in-addr.arpa.

130     IN      PTR     ns1.grp%GRP%.%DOMAIN%.
131     IN      PTR     ns2.grp%GRP%.%DOMAIN%.
```

Check the zone:

```
named-checkzone %GRP%.100.100.in-addr.arpa /var/lib/bind/zones/db.%GRP%.100.100.in-addr.arpa
```

Reload BIND:

```
rndc reload
```

# Configure the secondary servers

The new zone:

```
64-127.%GRP%.100.100.in-addr.arpa.
```

must also be served by `ns1` and `ns2`.

Configure both servers as secondary authoritative servers for the new zone, in the same way that you configured them as secondaries for the original reverse zone.

Check your configuration and reload BIND on both servers.

# Follow a classless reverse DNS query

Now let's see what happens when somebody performs a normal reverse lookup.

Query the original reverse zone:

```
dig @100.100.%GRP%.130 67.%GRP%.100.100.in-addr.arpa. PTR
```

Look carefully at the answer.

You should see a CNAME similar to:

```
67.%GRP%.100.100.in-addr.arpa. IN CNAME 67.64-127.%GRP%.100.100.in-addr.arpa.
```

The original reverse zone no longer contains the PTR record.

Instead, it tells the resolver to continue with another name.

Now query that name directly:

```
dig @100.100.%GRP%.130 67.64-127.%GRP%.100.100.in-addr.arpa. PTR
```

You should see:

```
67.64-127.%GRP%.100.100.in-addr.arpa. IN PTR resolv1.grp%GRP%.%DOMAIN%.
```

Now perform a normal reverse lookup using your recursive resolver:

```
dig -x 100.100.%GRP%.67
```

The resolver follows the CNAME automatically.

The complete lookup is therefore:

```
100.100.%GRP%.67
        |
        v
67.%GRP%.100.100.in-addr.arpa.
        |
        | CNAME
        v
67.64-127.%GRP%.100.100.in-addr.arpa.
        |
        | PTR
        v
resolv1.grp%GRP%.%DOMAIN%.
```

# Examine the delegation

Let's verify that `64-127` really is a delegated DNS zone.

Query for its NS records:

```
dig @100.100.%GRP%.130 64-127.%GRP%.100.100.in-addr.arpa. NS
```

Which name servers are authoritative for the zone?

Now check its SOA:

```
dig @100.100.%GRP%.130 64-127.%GRP%.100.100.in-addr.arpa. SOA
```

Compare this with:

```
dig @100.100.%GRP%.130 %GRP%.100.100.in-addr.arpa. SOA
```

Although both zones may currently use the same servers, they are two separate DNS zones and could be operated by different organizations.

# What about all the other addresses?

Our `/26` contains 64 addresses:

```
100.100.%GRP%.64 - 100.100.%GRP%.127
```

So far we created CNAMEs only for the addresses that we actually use:

```
66
67
68
```

In a production classless reverse delegation, the parent operator would normally create CNAME records for all addresses for which reverse DNS can be delegated.

For example:

```
65 IN CNAME 65.64-127.%GRP%.100.100.in-addr.arpa.
66 IN CNAME 66.64-127.%GRP%.100.100.in-addr.arpa.
67 IN CNAME 67.64-127.%GRP%.100.100.in-addr.arpa.
68 IN CNAME 68.64-127.%GRP%.100.100.in-addr.arpa.
69 IN CNAME 69.64-127.%GRP%.100.100.in-addr.arpa.
...
126 IN CNAME 126.64-127.%GRP%.100.100.in-addr.arpa.
```

The child operator can then create and remove PTR records in the delegated zone without requiring changes to the parent zone.

This separation of responsibility is the main reason for using RFC 2317.

# Test the complete configuration

Test the addresses in the delegated `/26`:

```
dig -x 100.100.%GRP%.66
dig -x 100.100.%GRP%.67
dig -x 100.100.%GRP%.68
```

They should return:

```
soa.grp%GRP%.%DOMAIN%.
resolv1.grp%GRP%.%DOMAIN%.
resolv2.grp%GRP%.%DOMAIN%.
```

Now test addresses outside the delegated `/26`:

```
dig -x 100.100.%GRP%.130
dig -x 100.100.%GRP%.131
```

They should still return:

```
ns1.grp%GRP%.%DOMAIN%.
ns2.grp%GRP%.%DOMAIN%.
```

The first three PTR records are now obtained through the classless reverse delegation.

The last two PTR records continue to come directly from the original reverse zone.

# Questions

Before continuing, make sure you can answer the following questions:

1. Why can `%GRP%.100.100.in-addr.arpa.` be delegated normally, while `100.100.%GRP%.64/26` cannot?

2. Is `64-127` special DNS syntax?

3. What record type is used in the parent reverse zone to redirect an individual reverse lookup into the classless reverse zone?

4. Where is the actual PTR record for `100.100.%GRP%.67` stored after the classless delegation is configured?

5. Who controls the CNAME records: the parent operator or the operator of the delegated network?

6. Who controls the PTR records inside `64-127.%GRP%.100.100.in-addr.arpa.`?

7. If the operator of the delegated network wants to change the PTR record for `100.100.%GRP%.67`, does the parent operator need to make a change?

8. Why do the PTR records for `100.100.%GRP%.130` and `100.100.%GRP%.131` remain in the original reverse zone?

# Summary

Normal IPv4 reverse DNS delegation works naturally when the network boundary corresponds to an octet boundary.

For example:

```
100.100.%GRP%.0/24
```

can use:

```
%GRP%.100.100.in-addr.arpa.
```

But a network such as:

```
100.100.%GRP%.64/26
```

does not have a corresponding natural DNS label boundary.

RFC 2317 solves this by combining two normal DNS mechanisms:

* an additional delegated DNS zone, and
* CNAME records in the parent reverse zone.

In our example:

```
67.%GRP%.100.100.in-addr.arpa.
```

is a CNAME for:

```
67.64-127.%GRP%.100.100.in-addr.arpa.
```

and that name contains the PTR record:

```
resolv1.grp%GRP%.%DOMAIN%.
```

This allows the operator of the `/26` network to manage its own PTR records even though the network does not end at an `in-addr.arpa` label boundary.

For more information, see RFC 2317, **Classless IN-ADDR.ARPA delegation**.
