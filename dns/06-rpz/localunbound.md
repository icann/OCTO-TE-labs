---
layout: page
title: "Local RPZ with Unbound"
parent: "RPZ"
nav_order: 6
---

# Local RPZ with Unbound

Sometimes you need to have local configuration for your RPZ. 
Maybe you need to override some things in a RPZ you subscribe to
or you want to block additional resources or maybe block 
some local clients.

# Setup

On the blocking resolver we need to configure our own RPZ.
```
sudo nano /etc/unbound/unbound.conf
```
Please add the following section **above** the old `rpz` section
```
rpz:
    name: "rpz-local."
    zonefile: "/var/lib/unbound/rpz-local.zone"
    rpz-log: yes
    rpz-log-name: "RPZ-LOCAL"
```
> [!NOTE] 
> The local zone needs to be configured before the other zones. RPZ will be
evaluated in the order given in this configuration.

Almost done, we just need to create our zone file
```
sudo touch /var/lib/unbound/rpz-local.zone
sudo chown -R unbound:unbound /var/lib/unbound
```
Now we add some content to the zone file
```
sudo nano /var/lib/unbound/rpz-local.zone
```
Please paste in the following content
```
$TTL    30
@   IN  SOA rpz-local. hostmaster.rpz.internal. (
        1      ; serial
        30     ; refresh
        30     ; retry
        30     ; expire
        30 )   ; minimum
@   IN  NS  localhost.

32.2.%GRP%.100.100.rpz-client-ip CNAME rpz-tcp-only.
drop.internal CNAME rpz-passthru.
```
Check if everything is configured correct
```
sudo unbound-checkconf /etc/unbound/unbound.conf
```
And then restart the resolver
```
sudo unbound-control reload
```
Now let's test if it is working
```
sudo unbound-control list_auth_zones
```
Should show the rpz zone like
```
rpz.	serial 1
```

# Local blocking and unblocking

## Client IP trigger
Our RPZ contains the following line
```
32.2.%GRP%.100.100.rpz-client-ip CNAME rpz-tcp-only.
```
The same IP address notation as before, prefix length and IP address in reverse order.
This matches the IP of the client sending queries.

Execute the following queries on your cli machine
```
dig google.com
```
Check the comments at the end of the dig output for which transport was used.
Our RPZ definition should have forced your client to use TCP. Do the same query on your 
resolv1 machine, it should use UDP.

## Overriding 
Remember in the RPZ `rpz` we subscribed the `drop.internal` is blocked. Any query will
be dropped. But in our RPZ `rpz-local` we defined an override, a passthru definition.
```
drop.internal CNAME rpz-passthru.
```
Again, query from your cli machine
```
dig drop.internal TXT
```
Now you should get an answer.
