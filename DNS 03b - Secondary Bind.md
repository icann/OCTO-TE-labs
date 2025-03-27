# Configure secondary server with Bind 9

The official Bind 9 configuration reference manual can be found at 
[https://bind9.readthedocs.io/en/latest/reference.html](https://bind9.readthedocs.io/en/latest/reference.html)

These servers are the ones that expose our zone publicly (so they will be open-to-all servers).

> [!IMPORTANT]
> Please check which one of the two secondaries ns1/ns2 you are 
> configuring and replace ***NS?*** in the following configuration 
> with the correct value.

# Server configuration

Create the directory and file that will contain our zone.

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX.secondary
$ sudo chown -R bind:bind /var/lib/bind
```

To do this, in the ***/etc/bind/named.conf.local*** file

```
sudoedit /etc/bind/named.conf.local
```

Configure the following parameters (replace \<server id\> with the hostname of the machine you are configuring)

```
server-id "<server id>";
version "grpX"
hostname;

zone "grpX.<lab domain>" {
    type secondary;
    file "/etc/bind/zones/db.grpX.secondary";
    masters { 
        100.100.X.66; 
        fd89:59e0:X:64::66;
        100.100.X.2; 
        fd89:59e0:X::2; 
    };
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

At he end of this section please proceed with [DNS 02 - Configure zone](DNS%2002%20-%20Configure%20zone.md#setting-up-the-secondaries)
