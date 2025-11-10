# Automatic zone signing

> [!WARNING]
> To change to automatic signing we will have to change keys. This can be done in two ways: 
> 1. With a Key Rollover, your domain stays signed all the time
> 2. With going insecure
>
> We will do the later in this step. (KSK Rollover is another lab)

## Going Insecure

Remove the DS record from the parent. Easiest done on the web page for your lab group. At the bottom you will find a button `Delete all DS records`.

## Revert to unsigned zone

> [!IMPORTANT]
> If you did the manual signing and confirm that your public nameservers are serving the signed zone, you should:
>
> 1. revert back `named.conf.local` to its previous configuration, i.e. configure BIND to serve the unsigned zone file as before the manual signing configuration which was: `file "/var/lib/bind/zones/db.grpX";` 
> 1. delete the signed zone file (.signed) BIND will create its own signed zone file in the next step.
> 1. increase the serial in the unsigned zone file and reload BIND.

## Edit config file.

Update your zone configuration statement in `/etc/bind/named.conf.local`, to look like the below : 

> [!WARNING] This DNSSEC policy configures extremly fast DNSSEC data changes for the purpose of running an efficient lab environment. This will break your domains when used in production. 

```
dnssec-policy NotForProduction {
    inline-signing yes;
    dnskey-ttl 5;
    max-zone-ttl 300;
    offline-ksk false;
    parent-ds-ttl 60s;
    parent-propagation-delay 1s;
    publish-safety 0s;
    purge-keys 1h;
    retire-safety 1m;
    signatures-jitter 31s;
    signatures-refresh 1m;
    signatures-validity 10m;
    signatures-validity-dnskey 2m;
    zone-propagation-delay 1s;
    keys {
        ksk key-directory lifetime unlimited algorithm ecdsa256;
        zsk key-directory lifetime unlimited algorithm ecdsa256;
    };
    parental-agents { };
    checkds no;
    cdnskey no;
    cds-digest-types { };
};

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
}; 
```

Then, reconfigure or restart BIND: using 
```
$ rndc reconfig
$ rndc reload
```

Check DNSSEC status of your zone:
```
$ rndc dnssec -status grpX.lab_domain
dnssec-policy: default
current time:  Wed May 28 14:14:13 2025

key: 64655 (ECDSAP256SHA256), CSK
  published:      yes - since Wed May 28 14:13:21 2025
  key signing:    yes - since Wed May 28 14:13:21 2025
  zone signing:   yes - since Wed May 28 14:13:21 2025

  No rollover scheduled
  - goal:           omnipresent
  - dnskey:         rumoured
  - ds:             hidden
  - zone rrsig:     rumoured
  - key rrsig:      rumoured
```

Some new files should appear in the *zones* directory.

## Use command line tools to query the signed zone.
We can now use *dig* utility to confirm that the zone is signed and play with the new DNSSEC RRs.

1. dig @100.100.X.130 grpX.lab_domain DNSKEY +dnssec
1. dig @100.100.X.131 grpX.lab_domain DNSKEY +dnssec +multi

Please verify:
1. How many keys did you get?
1. Are you sure your zone was updated?
1. Why?