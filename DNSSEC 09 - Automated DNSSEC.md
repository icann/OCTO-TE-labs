> [!WARNING]
> This is a configuration for this lab, it is absolutely unfit for use in any kind of real world deployment.

```
dnssec-policy lightspeed {
    inline-signing yes;
    dnskey-ttl 5;
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
    signatures-validity-dnskey 2m;
    zone-propagation-delay 1s;
    keys {
        ksk key-directory lifetime 15m algorithm ecdsa256;
        zsk key-directory lifetime 10m algorithm ecdsa256;
    };
    parental-agents { 100.100.X.67; fd89:59e0:X:64::68; };
    checkds no;
    cdnskey yes;
    cds-digest-types { SHA-256; };
};
```