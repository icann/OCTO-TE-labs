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
zone "grp%GRP%.%DOMAIN%." {
	type primary;
	file "/var/lib/bind/zones/db.grp%GRP%";
	allow-transfer { any; };
	also-notify {
		100.100.%GRP%.130; 
		100.100.%GRP%.131; 
		%IPv6pfx%:%GRP%:128::130; 
		%IPv6pfx%:%GRP%:128::131; 
	};
	dnssec-policy NotForProduction;
    parental-agents { 100.100.%GRP%.67; %IPv6pfx%:%GRP%:64::68; };
    checkds explicit;
}; 
```
```
named-checkconf
sudo rndc reload
```
There are now two new type of records in the zone CDS and CDNSKEY.
```
dig grp%GRP%.%DOMAIN% CDS
dig grp%GRP%.%DOMAIN% CDNSKEY
```
Again  try to onitor the status of the keys in the zone. This time we do not have to tell bind when the DS record is update.
