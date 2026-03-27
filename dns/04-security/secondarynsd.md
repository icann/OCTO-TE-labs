---
layout: page
title: "Secondary NSD"
parent: "Security"
nav_order: 4
---

# Secondary Security NSD

Currently our secondary server does not receive any zone updates.
This is due to the fact that we configured our primary to only allow
transfers with TSIG, but havent't configured TSIG on our secondaries.

## Add the TSIG key to your secondary NSD

Edit **/etc/nsd/nsd.conf** file, create key section and add the tsig key grpX-key

```
key:
        name: "grpX-key"
        algorithm: hmac-sha256
        secret: "THIS_IS_MY_KEY"
```

change these lines

```
allow-notify: 100.100.X.66 NOKEY
request-xfr: AXFR 100.100.X.66 NOKEY
```
with this:

```
allow-notify: 100.100.X.66 grpX-key
request-xfr: AXFR 100.100.X.66 grpX-key
```

Save, exit, verify and restart NSD service.

```
nsd-checkconf /etc/nsd/nsd.conf
sudo nsd-control reconfig
sudo nsd-control reload grpX.lab_domain
```

Check the logs on NS2 and on SOA.
