# Primary Setup with Bind9

The official Bind 9 configuration reference manual can be found at 
[https://bind9.readthedocs.io/en/latest/reference.html](https://bind9.readthedocs.io/en/latest/reference.html)

> [!IMPORTANT]
>
> In all this lab, be carefull to always replace ***X*** by your Group number in IP addresses, server name and any other place where required. Same for ***lab_domain*** to be replace by the domain name registered for the class.

## Install Bind 9

```
$ sudo apt -y install bind9
$ sudo adduser sysadm bind
```

This installs bind and allows our current user to use rndc to control bind.

## Setting the authoritative zone

We use the container "SOA" (hidden primary authoritative)

We create a new folder for our zone files. Inside that new folder, we then create a new file for our domain zone data.

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX
$ sudo chown -R bind:bind /var/lib/bind
```

Then, update the db.grp***X*** zone to look like the below:

```
; grpX 

$TTL    300
@       IN      SOA     soa.grpX.lab_domain. dnsadmin.grpX.lab_domain. (                                            
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                            300 )       ; Negative Cache TTL
@           NS          lab_domain.
@           TXT         "DNS IS FUN" 
ns1         A           100.100.X.130
ns1         AAAA        fd89:59e0:X:128::130
ns2         A           100.100.X.131
ns2         AAAA        fd89:59e0:X:128::131
```

You can add more records as you want.

In the configuration file ***/etc/bind/named.conf.local*** , create a new "zone" statement as below:

```
zone "grpX.lab_domain." {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { any; };
	also-notify {
		100.100.X.130; 
		100.100.X.131; 
		fd89:59e0:X:128::130; 
		fd89:59e0:X:128::131; 
	};
}; 
```

> [!TIP]
> Once done, use ***named-checkconf*** to verify that your BIND config is correct.
```
$ named-checkconf
```

Tell bind to reload the configuration and verify its status. You should see an output as the below
```
$ rndc reload
server reload successful
$ rndc zonestatus grpX.lab_domain
name: grpX.lab_domain
type: primary
files: /var/lib/bind/zones/db.grp1
serial: 1
nodes: 3
last loaded: Fri, 28 Mar 2025 14:19:54 GMT
secure: no
dynamic: no
reconfigurable via modzone: no
```

> [!IMPORTANT]
> One important aspect of using rndc to manage bind is, that bind tries to load the new 
> configuration and if something fails it continues to use the old configuration.

# Check Results

Query your zone on the local server:

```
$ dig @localhost soa grpX.lab_domain +noall + answer
grpX.lab_domain. 300 IN SOA grpX.lab_domain. dnsadmin.lab_domain. 1 604800 86400 2419200 300
```

To continue with the lab with [Setting up the secondaries](DNS%2003%20-%20Authoritative.md#setting-up-the-secondaries)
