# Configure secondary server with NSD

> [!TIP]
> The official NSD configuration reference manual can be found at
> [https://nsd.docs.nlnetlabs.nl/en/latest/index.html](https://nsd.docs.nlnetlabs.nl/en/latest/index.html)

> [!IMPORTANT]
> Please check which one of the two secondaries ns1/ns2 you are 
> configuring and replace ***NS?*** in the following configuration 
> with the correct value.

# Server configuration

Create the directory and file that will contain our zone file.

```
$ sudo mkdir -p /var/lib/nsd
$ sudo touch /var/lib/nsd/db.grpX.secondary
$ chown -R nsd:nsd /var/lib/nsd
```

In the ***/etc/nsd/nsd.conf*** file, configure the following parameters:

```
include: "/etc/nsd/nsd.conf.d/*.conf"

server:
	zonesdir: "/var/lib/nsd"
    nsid: "NS?"
    hide-version: no
    hide-identity: no
    
pattern:
	name: "fromprimary"
	allow-notify: 100.100.X.66 NOKEY
    allow-notify: fd89:59e0:X:64::66 NOKEY
    allow-notify: fd89:59e0:X::2 NOKEY
	request-xfr: AXFR 100.100.X.66 NOKEY
	request-xfr: AXFR fd89:59e0:X:64::66 NOKEY
    request-xfr: AXFR fd89:59e0:X::2 NOKEY

zone:
	name: "grpX.<lab domain>."
	zonefile: "db.grpX.secondary"
	include-pattern: "fromprimary"
```

Verify the configuration and if there are no errors restart the server:

```
$ nsd-checkconf /etc/nsd/nsd.conf
$ nsd-control reconfig
```

Verify that it restarted correctly:

```
$ nsd-control status
$ nsd-control zonestatus grpX.<lab domain>
```

At he end of this section please proceed with [DNS 02 - Configure zone](DNS%2002%20-%20Configure%20zone.md#setting-up-the-secondaries)
