---
layout: page
title: "Bind"
parent: "Automated DNSSEC"
nav_order: 1
---

# Automated DNSSEC with Bind

> [!WARNING]
> To change to automatic signing we will have to change keys. This can be done in two ways: 
> 1. With a Key Rollover, your domain stays signed all the time
> 2. With going insecure
>
> We will do the later in this step. 

## Going Insecure

Remove the DS record from the parent. Easiest done on the web page for your lab group. At the bottom you will find a button `Delete all DS records`.

## Configuration
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
```
```
sudo rndc reload
```
There are now two new type of records in the zone CDS and CDNSKEY.
```
dig grp%GRP%.%DOMAIN% CDS
dig grp%GRP%.%DOMAIN% CDNSKEY
```
Again  try to monitor the status of the keys in the zone. This time we do not have to tell bind when the DS record is update.
