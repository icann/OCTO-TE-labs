# Signing

In this lab we will use the signing method with two keys, a Key-Signing-Key (KSK) and a Zone-Signing-Key (ZSK).

The KSK signs the DNSKEY set (and potential CDS / CDNSKEY / CSYNC records), the ZSK signs all other data.


To sign the zone we first need two pairs of keys: a ZSK and a KSK. 


Position yourself in BIND configuration folder and then backup your zone file:

```
$ sudo cp /var/lib/bind/zones/db.grpX /var/lib/bind/zones/db.grpX.backup
```

Create a directory to hold your DNSSEC keys

```
$ sudo mkdir -p /var/lib/bind/keys
```

Generate the **ZSK**

```
$ sudo dnssec-keygen -f ZSK -a ECDSAP256SHA256 -K /var/lib/bind/keys grpX.lab_domain
```

Generate **KSK**

```
$ sudo dnssec-keygen -f KSK -a ECDSAP256SHA256 -K /var/lib/bind/keys grpX.lab_domain
```

Change ownership to the zones and keys folders

```
$ sudo chown -R bind:bind /var/lib/bind/keys
```

# Manual zone signing.

We start with manual zone signing.

> [!IMPORTANT] Don't do this in production! This lab is meant to give you an impression of what is going on behind the scenes when you use the automation.

```
$ sudo dnssec-signzone -S -K /var/lib/bind/keys -o grpX.lab_domain /var/lib/bind/zones/db.grpX
```

You should get an output similar to the follwing:

```
Fetching grpX.lab_domain/ECDSAP256SHA256/61520 (KSK) from key repository.
Fetching grpX.lab_domain/ECDSAP256SHA256/12593 (ZSK) from key repository.
Verifying the zone using the following algorithms:
- ECDSAP256SHA256
Zone fully signed:
Algorithm: ECDSAP256SHA256: KSKs: 1 active, 0 stand-by, 0 revoked
                            ZSKs: 1 active, 0 stand-by, 0 revoked
/var/lib/bind/zones/db.grpX.signed
```

Now, replace the *db.grpX* file in `named.conf.local` with the `db.grpX.signed` and reload the server using 

```rndc reload```.

# Verify the configuration

We can now use *dig* utility to confirm that the zone is signed and play with the new DNSSEC RRs.

```
$ dig @localhost soa grpX.lab_domain +dnssec 
```

This should give you an output similiar to
```
; <<>> DiG 9.16.1-Ubuntu <<>> @localhost soa grpX.lab_domain. +dnssec
; (2 servers found)                                                               
;; global options: +cmd                                                           
;; Got answer:                                                                  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9591                         
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1                                                                   
;; OPT PSEUDOSECTION:                                                             
; EDNS: version: 0, flags: do; udp: 4096
; COOKIE: 69a0c61239afd9a201000000609c5df711d4eb3a39f90d89 (good)
;; QUESTION SECTION:
;grpX.lab_domain.        IN      SOA 

;; ANSWER SECTION:
grpX.lab_domain. 30 IN   SOA     soa.grpX.lab_domain. dnsadmin.grpX.lab_domain. 1 604800 86400 2419200 300
grpX.lab_domain. 30 IN   RRSIG   SOA 8 4 30 20210611215606 20210512215606 41110 grpX.lab_domain. RmUb[...]=
```

**QUESTION**: Did you get the "ad" flag? Why?

More tests: 

1. dig @100.100.X.130 grpX.lab_domain SOA
1. dig @100.100.X.130 grpX.lab_domain DNSKEY +dnssec +multi
1. dig @100.100.X.67  grpX.lab_domain SOA
1. dig @100.100.X.68  grpX.lab_domain DNSKEY +dnssec +multi

Please discuss with your peers and the instructor: 

1. Did you get the answers? 
1. Did you receive signatures?
1. Did you get the "ad" flag? Why?
