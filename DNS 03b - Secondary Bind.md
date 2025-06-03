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

> [!TIP]
> Close and reopen your shell window. The new user permissions only get active after logging out and in again.

# Server configuration

Create the directory and file that will contain our zone.

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX.secondary
$ sudo chown -R bind:bind /var/lib/bind
```

Configure the server as secondary for our domain grpX.lab_domain.

To do this we edit the bind configuration

```
$ sudo nano /etc/bind/named.conf.local
```

Change the file contents to

```
zone "grpX.lab_domain" {
    type secondary;
    file "/etc/bind/zones/db.grpX.secondary";
    masters { 
        100.100.X.66; 
        fd89:59e0:X:64::66;
    };
};
```

Configure bind options.

```
$ sudo nano /etc/bind/named.conf.options
```

> Please replace ***server_id*** and ***host_name*** with something unique - it's ok to be creative

```
options {
    directory "/var/cache/bind";
    server-id "server_id";
    version "grpX";
    hostname "host_name";
    dnssec-validation no;
    listen-on port 53 { localhost; 100.100.0.0/16; };
    listen-on-v6 port 53 { localhost; fd89:59e0::/32; };
    allow-query { any; };
    recursion yes;
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
