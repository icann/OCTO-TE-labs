# Zone Delivery

Zone generation and zone signing are very critical operations. Any error can 
have huge consequences. Therefore do most operators of critical zones use a 
technic called zone pipeline. It is simply a sequences of steps, where later 
steps check if earlier steps succeeded and did the right thing.

1. Check zone file for completeness
1. Validate all signatures
1. Check NSEC or NSEC3 chains

In our lab we will only use a very simple mock-up script for checking a zone.

# Configuration of hidden primary

Currently the primary delivers the zone directly to the secondaries. We will
use NSD to implement an additional checking step.

We start with changing `/etc/bind/named.conf.options` to
```
options {
    directory "/var/cache/bind";
    server-id "hidden primary";
    listen-on port 5353 { localhost; 100.100.0.0/16; };
    listen-on-v6 port 5353 { localhost; fd89:59e0::/32; };
    allow-query { any; };
    recursion no;
};
```
And the `/etc/bind/named.conf.local`file to
```
zone "grpX.lab_domain." {
    type primary;
    file "/var/lib/bind/zones/db.grpX";
    allow-transfer { ::1; };
    also-notify {
        ::1;
    };
    dnssec-policy default;
};
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
    nsid: "ascii_grpX zone validator"
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
    notify: 100.100.X.130 NOKEY
    notify: 100.100.X.131 NOKEY
    notify: fd89:59e0:X:128::130 NOKEY 
    notify: fd89:59e0:X:128::131 NOKEY
    provide-xfr: 100.100.X.130 NOKEY
    provide-xfr: 100.100.X.131 NOKEY
    provide-xfr: fd89:59e0:X:128::130 NOKEY 
    provide-xfr: fd89:59e0:X:128::131 NOKEY

zone:
    name: "grpX.lab_domain."
    zonefile: "db.grpX.secondary"
    include-pattern: "fromprimary"
```
Now we need to make a small zone checking script. Please edit the file `/var/lib/nsd/test.sh` with
```
$ sudo nano /var/lib/nsd/test.sh
```
to the following content
```
#!/bin/bash
exit 5;
```
And make it executable with 
```
$ sudo chmod +x /var/lib/nsd/test.sh
$ sudo chown nsd:nsd /var/lib/nsd/test.sh
```
 
Ready to restart

1. Open another shell window on the soa machine
1. Run `sudo tail -f /var/log/syslog`
1. Back to the first shell window
1. Restart Bind `sudo systemctl restart named`
1. Restart NSD `sudo systemctl restart nsd`
1. Edit the zone file `/var/lib/bind/zones/db.grpX`, increase the serial number
1. Run `rndc reload`
1. Back to the new shell window and see if you can identify the validation in the logs

Lets check what contents our servers carry.

1. `dig @localhost -p 5353    grpX.lab_domain SOA +nsid`
1. `dig @localhost -p 53      grpX.lab_domain SOA +nsid`
1. `dig @100.100.X.130        grpX.lab_domain SOA +nsid`
1. `dig @fd89:59e0:X:128::131 grpX.lab_domain SOA +nsid`

Did all servers show the correct serial number?
Did you get different id strings for all servers?

Currently our script fails all zones. Let's try if we approve all zones.

1. Edit `/var/lib/nsd/test.sh` again and change `exit 5` to `exit 0`.
1. Edit the zone file `/var/lib/bind/zones/db.grpX`, increase the serial number
1. Run `rndc reload`
1. Back to the new shell window and see if you can identify the validation in the logs

What's different?

Run the same dig commands again:

1. `dig @localhost -p 5353    grpX.lab_domain SOA +nsid`
1. `dig @localhost -p 53      grpX.lab_domain SOA +nsid`
1. `dig @100.100.X.130        grpX.lab_domain SOA +nsid`
1. `dig @fd89:59e0:X:128::131 grpX.lab_domain SOA +nsid`

Did all servers show the correct serial number?
Did you get different id strings for all servers?
