---
layout: page
title: "Bind"
parent: "Automatic Signing"
nav_order: 1
---

# Automatic zone signing

> [!WARNING]
> To change to automatic signing we will have to change keys. This can be done in two ways: 
> 1. With a Key Rollover, your domain stays signed all the time
> 2. With going insecure
>
> We will do the later in this step. 

## Going Insecure

Remove the DS record from the parent. Easiest done on the web page for your lab group. At the bottom you will find a button `Delete all DS records`.

## Revert to unsigned zone

> [!IMPORTANT]
> If you did the manual signing and confirm that your public nameservers are serving the signed zone, you should:
>
> 1. revert back `named.conf.local` to its previous configuration, i.e. configure BIND to serve the unsigned zone file as before the manual signing configuration which was: `file "/var/lib/bind/zones/db.grp%GRP%";` 
> 1. delete the signed zone file (/var/lib/bind/zones/db.grp%GRP%.signed) BIND will create its own signed zone file in the next step.
> 1. increase the serial in the unsigned zone file and reload BIND.

## Edit config file.

Update your zone configuration statement in `/etc/bind/named.conf.local`, to look like the below : 

> [!WARNING] 
> This DNSSEC policy configures extremly fast DNSSEC data changes for the purpose of 
> running an efficient lab environment. This will break your domains when used in production. 

> [!TIP] 
> Bind has a pre-configured policy that you absolutely should consider using instead of defining your own just use `dnssec-policy default`.

```
dnssec-policy NotForProduction {
    inline-signing yes;
    dnskey-ttl 5;
    max-zone-ttl 300;
    offline-ksk false;
    parent-ds-ttl 60s;
    parent-propagation-delay 1s;
    zone-propagation-delay 1s;
    publish-safety 0s;
    purge-keys 1h;
    retire-safety 1m;
    signatures-jitter 31s;
    signatures-refresh 1m;
    signatures-validity 10m;
    signatures-validity-dnskey 2m;
    keys {
        ksk key-directory lifetime unlimited algorithm ecdsa256;
        zsk key-directory lifetime unlimited algorithm ecdsa256;
    };
    cdnskey no;
    cds-digest-types { };
};

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
    checkds no;
}; 
```

Then, reconfigure or restart BIND: using 
```
sudo rm /var/lib/bind/zones/db.grp%GRP%.signed
sudo rndc reconfig
sudo rndc reload
```

Check DNSSEC status of your zone:
```
sudo rndc dnssec -status grp%GRP%.%DOMAIN%
```
```
dnssec-policy: NotForProduction
current time:  Tue Apr 14 17:17:02 2026

key: 15345 (ECDSAP256SHA256), KSK
  published:      yes - since Tue Apr 14 09:26:02 2026
  key signing:    yes - since Tue Apr 14 09:26:02 2026

  No rollover scheduled
  - goal:           omnipresent
  - dnskey:         omnipresent
  - ds:             omnipresent
  - key rrsig:      omnipresent

key: 15717 (ECDSAP256SHA256), ZSK
  published:      yes - since Tue Apr 14 07:33:10 2026
  zone signing:   yes - since Tue Apr 14 07:33:10 2026

  No rollover scheduled
  - goal:           omnipresent
  - dnskey:         omnipresent
  - zone rrsig:     omnipresent

```

Some new files should appear in the *zones* directory.

## Use command line tools to query the signed zone.
We can now use *dig* utility to confirm that the zone is signed and play with the new DNSSEC RRs.

1. dig @100.100.%GRP%.130 grp%GRP%.%DOMAIN% DNSKEY +dnssec
1. dig @100.100.%GRP%.131 grp%GRP%.%DOMAIN% DNSKEY +dnssec +multi

Please verify:
1. How many keys did you get?
1. Are you sure your zone was updated?
1. Why?