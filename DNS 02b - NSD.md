#### Configure ns2 [**ns2.grpX**] server

**Server ns2 runs NSD** (from NLnet Labs)

To do this, in the ***/etc/nsd/nsd.conf*** file, configure the following parameters:

```
# NSD configuration file for Debian.
#
# See the nsd.conf(5) man page.
#
# See /usr/share/doc/nsd/examples/nsd.conf for a commented
# reference config file.
#
# The following line includes additional configuration files from the
# /etc/nsd/nsd.conf.d directory.


include: "/etc/nsd/nsd.conf.d/*.conf"

server:
	zonesdir: "/var/lib/nsd"
    nsid: "ns2"
    hide-version: no
    version: "grpX"
    hise-identity: no
    #identity:

pattern:
	name: "fromprimary"
	allow-notify: 100.100.X.66 NOKEY
    allow-notify: fd73:7c99:X::2 NOKEY
	request-xfr: AXFR 100.100.X.66 NOKEY
    request-xfr: IXFR fd73:7c99:X::2 NOKEY

zone:
	name: "grpX.<lab_domain>.te-labs.training"
	zonefile: "grpX.<lab_domain>.te-labs.training.zone"
	include-pattern: "fromprimary"
```

Verify the configuration and if there are no errors restart the server:

```
# nsd-checkconf /etc/nsd/nsd.conf
# nsd-control reconfig
```

Verify that it restarted correctly:

```
# nsd-control status
```

```
```

