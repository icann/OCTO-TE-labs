# Primary Setup with Bind9

The official Bind 9 configuration reference manual can be found at 
[https://bind9.readthedocs.io/en/latest/reference.html](https://bind9.readthedocs.io/en/latest/reference.html)

> [!IMPORTANT]
>
> In all this lab, be carefull to always replace ***X*** by your Group number in IP addresses, server name and any other place where required. Same for \<lab domain\> to be replace by the domain name registered for the class.

## Setting the authoritative zone

We use the container "SOA" (hidden primary authoritative)

We create a new folder for our zone files. Inside that new folder, we then create a new file for our domain zone data.

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX
$ sudo chown -R bind:bind /var/lib/bind
```

Then, update the db.grpX zone to look like the below:

```
; grpX 

$TTL    300
@       IN      SOA     soa.grpX.<lab domain>. dnsadmin.grpX.<lab domain>. (                                            
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                            300 )       ; Negative Cache TTL
@           NS          <lab domain>.
@           TXT         "DNS IS FUN" 
ns1         A           100.100.X.130
ns1         AAAA        fd89:59e0:X:128::130
ns2         A           100.100.X.131
ns2         AAAA        fd89:59e0:X:128::131
```

You can add more records as you want.

In the configuration file ***/etc/bind/named.conf.local*** , create a new "zone" statement as below:

```
zone "grpX.<lab domain>." {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { ns1; ns2; };
	also-notify {100.100.X.130; fd89:59e0:X:128::131; };
}; 
```

Tell bind to reload the configuration and verify its status. You should see an output as the below

```
$ rndc reload

OUTPUT

$ rndc zonestatus grpX.<lab domain>

OUTPUT
```

One important aspect of using rndc to manage bind is, that bind tries to load the new 
configuration and if something fails it continues to use the old configuration.

Now, let's fix our configuration

```
In the configuration file ***/etc/bind/named.conf.local*** , change the "zone" statement as below:
(pay attention to the `allow-transfer` statement)
```
zone "grpX.<lab domain>." {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { any; };
	also-notify {100.100.X.130; 100.100.X.131; fd89:59e0:X:128::130; fd89:59e0:X:128::131; };
}; 
```

> [!TIP]
> Once done, use ***named-checkconf*** to verify that your BIND config is correct.

```
$ sudo named-checkconf
```
Tell bind to reload the configuration and verify its status. You should see an output as the below

```
$ rndc reload
$ rndc zonestatus grpX.<lab domain>

OUTPUT
```

Then, query your zone on the local server:

```
$ dig @localhost soa grpX.<lab domain> +noall + answer
;; ANSWER SECTION:
grpX.<lab domain>. 30 IN   SOA     grpX.<lab domain>. dnsadmin.<lab domain>. 1 604800 86400 2419200 300
```

