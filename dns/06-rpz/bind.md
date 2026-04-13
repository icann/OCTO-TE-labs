---
layout: page
title: "Bind"
parent: "RPZ"
nav_order: 2
---

# RPZ with Bind

We have already setup a RPZ. You "just" need to configure it in your resolver.

On the resolv1 machine the resolver needs to be configured to use the RPZ.
```
sudo nano /etc/bind/named.conf.local
```
Please add the following section
```
zone "rpz" {
    type secondary;
    file "/var/lib/bind/zones/db.rpz.secondary";
    masters { 
        100.64.0.54; 
        %IPv6pfx%::54;
    };
    allow-transfer { none; };
    allow-query { localhost; };
};
```
And to start blocking we need to configure the use of the zone as RPZ.
```
sudo nano /etc/bind/named.conf.options
```
Please add the following logging section to the file
```
logging {
    channel rpzlog {
      	file "/var/log/named/rpz.log" versions unlimited size 100m;
        print-time yes;
        print-category yes;
        print-severity yes;
        severity info;
    };

	category rpz { rpzlog; };
};
```
And in the already existing options section add
```
    response-policy	{
        zone "rpz" ede blocked policy given;
    };
```
Almost done, we just need to prepare for the zone file transfer
```
sudo mkdir -p /var/lib/bind/zones
sudo touch /var/lib/bind/zones/db.rpz.secondary
sudo chown -R bind:bind /var/lib/bind
sudo mkdir -p /var/log/named
sudo chmod 775 /var/log/named
sudo chown -R root:bind /var/log/named
```
Check if everything is configured correct
```
named-checkconf
```
And then restart the resolver
```
sudo rndc reconfig
sudo rndc reload
```
Now let's test if it is working
```
dig @localhost rpz soa +nocomments +noall +answer
```
Should show you a SOA record.
