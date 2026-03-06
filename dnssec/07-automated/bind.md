---
layout: page
title: "Bind"
parent: "Automated DNSSEC"
nav_order: 1
---

> [!WARNING]
> This is a configuration for this lab, it is absolutely unfit for use in any kind of real world deployment.

```
sudo nano /etc/bind/named.conf.local
```
Please change the following values in the dnssec-policy to
```
    cdnskey yes;
    cds-digest-types { SHA-256; };
```
and in the same file change the configuration for your zone to
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
	dnssec-policy NotForProduction;
    parental-agents { 100.100.X.67; fd89:59e0:X:64::68; };
    checkds explicit;
}; 
```
```
named-checkconf
sudo rndc reload
```
There are now two new type of records in the zone CDS and CDNSKEY.
```
dig grpX.lab_domain CDS
dig grpX.lab_domain CDNSKEY
```
Again  try to onitor the status of the keys in the zone. This time we do not have to tell bind when the DS record is update.
