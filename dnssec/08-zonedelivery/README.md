---
layout: page
title: "Zone Delivery"
parent: "DNSSEC"
nav_order: 8
---

# Zone Delivery

Zone generation and zone signing are very critical operations. Any error can 
have huge consequences. Therefore do most operators of critical zones use a 
technic called zone pipeline. It is simply a sequences of steps, where later 
steps check if earlier steps succeeded and did the right thing.

1. Check zone file for completeness
1. Validate all signatures
1. Check NSEC or NSEC3 chains

Currently the primary delivers the zone directly to the secondaries. We will
use NSD to implement an additional checking step.

In our lab we will only use a very simple mock-up script for checking a zone.

# Configuration of hidden primary

On our soa machine we change the bind configuration to use a non-standard port 
and only on the localhost interface.

We start with changes to  `/etc/bind/named.conf.options`
```
    listen-on port 5353 { localhost; };
    listen-on-v6 port 5353 { localhost; };
```
And in file `/etc/bind/named.conf.local`
```
    allow-transfer { ::1; };
    also-notify { ::1; };
```

# Install NSD checking server

Please install NSD on the machine by running `sudo apt install -y nsd`.

Now we need to configure NSD:
```
include: "/etc/nsd/nsd.conf.d/*.conf"

server:
    zonesdir: "/var/lib/nsd"
    hide-version: no
    hide-identity: no
    nsid: "ascii_grp%GRP% zone validator"
    verbosity: 2

verify:
    enable: yes
    verify-zones: yes
    verifier-timeout: 10  
  
pattern:
    name: "fromprimary"
    allow-notify: ::1 NOKEY
    request-xfr: AXFR ::1@5353 NOKEY
    verify-zone: yes
    verifier: /var/lib/nsd/test.sh
    notify: 100.100.%GRP%.130 NOKEY
    notify: 100.100.%GRP%.131 NOKEY
    notify: %IPv6pfx%:%GRP%:128::130 NOKEY 
    notify: %IPv6pfx%:%GRP%:128::131 NOKEY
    provide-xfr: 100.100.%GRP%.130 NOKEY
    provide-xfr: 100.100.%GRP%.131 NOKEY
    provide-xfr: %IPv6pfx%:%GRP%:128::130 NOKEY 
    provide-xfr: %IPv6pfx%:%GRP%:128::131 NOKEY

zone:
    name: "grp%GRP%.%DOMAIN%."
    zonefile: "db.grp%GRP%.secondary"
    include-pattern: "fromprimary"
```
Now we need to make a small zone checking script. Please edit the file `/var/lib/nsd/test.sh` with
```
sudo nano /var/lib/nsd/test.sh
```
to the following content
```
#!/bin/bash
exit 5;
```
And make it executable with 
```
sudo chmod +x /var/lib/nsd/test.sh
sudo chown nsd:nsd /var/lib/nsd/test.sh
```
 
Ready to restart

1. Open another shell window on the soa machine
1. Run `sudo tail -f /var/log/syslog`
1. Back to the first shell window
1. Restart Bind `sudo systemctl restart named`
1. Restart NSD `sudo systemctl restart nsd`
1. Edit the zone file `/var/lib/bind/zones/db.grp%GRP%`, increase the serial number
1. Run `rndc reload`
1. Back to the new shell window and see if you can identify the validation in the logs

Lets check what contents our servers carry.

1. `dig @localhost -p 5353    grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @localhost -p 53      grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @100.100.%GRP%.130        grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @%IPv6pfx%:%GRP%:128::131 grp%GRP%.%DOMAIN% SOA +nsid`

Did all servers show the correct serial number?
Did you get different id strings for all servers?

Currently our script fails all zones. Let's try if we approve all zones.

1. Edit `/var/lib/nsd/test.sh` again and change `exit 5` to `exit 0`.
1. Edit the zone file `/var/lib/bind/zones/db.grp%GRP%`, increase the serial number
1. Run `rndc reload`
1. Back to the new shell window and see if you can identify the validation in the logs

What's different?

Run the same dig commands again:

1. `dig @localhost -p 5353    grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @localhost -p 53      grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @100.100.%GRP%.130        grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @%IPv6pfx%:%GRP%:128::131 grp%GRP%.%DOMAIN% SOA +nsid`

Did all servers show the correct serial number?
Did you get different id strings for all servers?
