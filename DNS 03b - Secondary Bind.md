# Configure secondary server with Bind 9

The official Bind 9 configuration reference manual can be found at 
[https://bind9.readthedocs.io/en/latest/reference.html](https://bind9.readthedocs.io/en/latest/reference.html)

These servers are the ones that expose our zone publicly (so they will be open-to-all servers).

# Install Bind 9

```
$ sudo apt -y install bind9
$ sudo adduser sysadm bind
```

This installs bind and allows our current user to use rndc to control bind.

# Server configuration

Create the directory and file that will contain our zone.

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX.secondary
$ sudo chown -R bind:bind /var/lib/bind
```

To do this, in the ***/etc/bind/named.conf.local*** file

```
$ sudo nano /etc/bind/named.conf.local
```

> Please check which one of the two secondaries ns1/ns2 you are 
> configuring and replace ***NS?*** in the following configuration 
> with the correct value.

> Replace ***server_ id*** with a unique id - it's ok to be creative

```
server-id "server_id";
version "grpX"
hostname;

zone "grpX.lab_domain" {
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
$ rndc zonestatus grpX.lab_domain 
```

# Check if the instance is working

The following two dig commands should produce the same result
```
$ dig @100.100.X.66 grpX.lab_domain SOA +noall +answer
$ dig @localhost grpX.lab_domain SOA +noall +answer
``` 

# Done

Congratulations! You Secondary name server instance is configured
Please proceed with [DNS 02 - Configure zone](DNS%2002%20-%20Configure%20zone.md#setting-up-the-secondaries)
