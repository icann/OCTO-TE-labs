# Configure your DNS zone

The official Bind 9 configuration reference manual can be found at 
[https://bind9.readthedocs.io/en/latest/reference.html](https://bind9.readthedocs.io/en/latest/reference.html)

> [!IMPORTANT]
>
> In all this lab, be carefull to always replace ***X*** by your Group number in IP addresses, server name and any other place where required. Same for \<lab domain\> to be replace by the domain name registered for the class.

## Configure primary authoritative server (BIND)

### Intro

We are going to configure a hidden authoritative server and create the authoritative zone **grpX.\<lab domain\>**.

### What we already know

Our "parent" (**\<lab domain\>**) has already created the following in its own zone:

```
; grpX
grpX             NS          <lab domain>.
; ---Placeholder for grpX DS record (DO NOT MANUALLY EDIT THIS LINE)---
ns1.grpX         A           100.100.X.130
ns1.grpX         AAAA        fd73:7c99:X:128::130
ns2.grpX         A           100.100.X.131
ns2.grpX         AAAA        fd73:7c99:X:128::131

```

Our zone configuration must be compatible with that.

### Setting the authoritative zone

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
@       IN      SOA     soa.grpX.<lab_domain>. dnsadmin.grpX.<lab_domain>. (                                            
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;

; grpX 
@             NS           <lab domain>.

ns1         A           100.100.X.130
ns1         AAAA        fd73:7c99:X:128::130
ns2         A           100.100.X.131
ns2         AAAA        fd73:7c99:X:128::131
www         A           100.100.X.130
```

You can add more records as you want.

> [!TIP]
>
> Once done, it is important to verify the zone file format. Use ***named-checkzone*** command with appropriate parameters to do that.

In the configuration file ***/etc/bind/named.conf.local*** , create a new "zone" statement as below:

```
zone "grpX.<lab_domain>." {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { any; };
	also-notify {100.100.X.130; fd73:7c99:X:128::131; };
}; 
```

> [!TIP]
> Once done, use ***named-checkconf*** to verify that your BIND config is correct.

Restart the DNS service and verify its status. You should see an output as the below

```
$ rndc reload
```

Then, query your zone on the local server:

```
$ dig @localhost soa grpX.<lab domain>
; <<>> DiG 9.16.1-Ubuntu <<>> @localhost soa grpX.<lab domain>.
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64339
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 270e2c46ed443c1c01000000609c59f04ba85015ff71998d (good)
;; QUESTION SECTION:
;grpX.<lab_domain>.te-labs.training.        IN      SOA

;; ANSWER SECTION:
grpX.<lab domain>. 30 IN   SOA     grpX.<lab domain>. dnsadmin.<lab domain>. 1 604800 86400 2419200 86400

;; Query time: 0 msec
;; SERVER: ::1#53(::1)
;; WHEN: Wed May 12 22:42:56 UTC 2021
;; MSG SIZE  rcvd: 170
```

