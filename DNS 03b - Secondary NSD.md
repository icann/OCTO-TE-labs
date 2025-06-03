# Configure secondary server with NSD

> [!TIP]
> The official NSD configuration reference manual can be found at
> [https://nsd.docs.nlnetlabs.nl/en/latest/index.html](https://nsd.docs.nlnetlabs.nl/en/latest/index.html)

> [!IMPORTANT]
> Please check which one of the two secondaries ns1/ns2 you are 
> configuring and replace ***NS?*** in the following configuration 
> with the correct value.

# Install NSD

```
$ sudo apt -y install nsd
```

This installs bind and allows our current user to use rndc to control bind.

Create the directory and file that will contain our zone file.

```
$ sudo mkdir -p /var/lib/nsd
$ sudo touch /var/lib/nsd/db.grpX.secondary
$ sudo chown -R nsd:nsd /var/lib/nsd
```

In the ***/etc/nsd/nsd.conf*** file, configure the following parameters:

```
include: "/etc/nsd/nsd.conf.d/*.conf"

server:
    zonesdir: "/var/lib/nsd"
    nsid: "ascii_grpX NSD nsid"
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
	name: "grpX.lab_domain."
	zonefile: "db.grpX.secondary"
	include-pattern: "fromprimary"
```

Verify the configuration and if there are no errors restart the server:

```
$ nsd-checkconf /etc/nsd/nsd.conf
$ sudo systemctl restart nsd
```

Verify that it restarted correctly:

```
$ sudo nsd-control status
$ sudo nsd-control zonestatus grpX.lab_domain
```
