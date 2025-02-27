## Configure secondary authoritative servers

These servers are the ones that expose our zone publicly (so they will be open-to-all servers).

#### Configure ns1 [**ns1.grpX**] server

**Server ns1 runs BIND** (from ISC)

Go to the `/etc/bind` directory and create a file that will contain our zone file in the nameserver:

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX.secondary
$ chown -R bind:bind /var/lib/bind
```

To do this, in the ***/etc/bind/named.conf.local*** file, configure the following parameters:

```
server-id "ns1";
version "grpX"
hostname;

zone "grpX.<lab domain>" {
    type secondary;
    file "/etc/bind/zones/db.grpX.secondary";
    masters { 100.100.X.66; fd73:7c99:X::2; };
};
```

Verify the configuration and if there are no errors, restart the server:

```
$ named-checkconf
$ rndc reload
```

Verify that it restarted correctly:

```
$ rndc zonestatus grpX.<lab domain> 
```

```
```

