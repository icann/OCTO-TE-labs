> [!WARNING]
> This is a configuration for this lab, it is absolutely unfit for use in any kind of real world deployment.

```
dnssec-policy lightspeed {
    cdnskey yes;
    cds-digest-types { SHA-256; };
    dnskey-ttl 5;
    inline-signing yes;
    keys {
        ksk key-directory lifetime 15m algorithm ecdsa256;
        zsk key-directory lifetime 10m algorithm ecdsa256;
    };
    max-zone-ttl 3600;
    offline-ksk false;
    parent-ds-ttl 60s;
    parent-propagation-delay 1s;
    publish-safety 0s;
    purge-keys 1h;
    retire-safety 1m;
    signatures-jitter 31s;
    signatures-refresh 1m;
    signatures-validity 10m;
    signatures-validity-dnskey 1m;
    zone-propagation-delay 1s;
};
```