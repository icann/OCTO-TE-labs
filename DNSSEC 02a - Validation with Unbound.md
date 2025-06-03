# Set up an Unbound recursive server

Follow this lab on the same machine you installed the Unbound resolver in Lab DNS 01.

Our current configuration in `/etc/unbound/unbound.conf` should be
```
server:
    interface: 0.0.0.0
    interface: ::0

    access-control: 0.0.0.0/0 allow
    access-control: ::/0 allow

    port: 53

    do-udp: yes
    do-tcp: yes
    do-ip4: yes
    do-ip6: yes

remote-control:
    control-enable: yes
```

In this case, there is no obvious configuration that stops DNSSEC validation.
But remember, what is needed for a resolver to to DNSSEC validation?

So we do need to add configuration for the trust-anchor.

In the server section of the configuration we need to add:
```
    auto-trust-anchor-file: /var/lib/unbound/root.key
```

Save the changes and run `unbound-checkconf`
```
$ sudo unbound-checkconf /etc/unbound/unbound.conf
/var/lib/unbound/root.key: No such file or directory
unbound-checkconf[13615:0] fatal error: auto-trust-anchor-file: "/var/lib/unbound/root.key" does not exist
```

We added the configuration, but we still do not have a trust-anchor.

Lukily Ubuntu does provide one for us:
```
$ sudo cp /usr/share/dns/root.key /var/lib/unbound
```
Let's check the configuration again
```
$ sudo unbound-checkconf /etc/unbound/unbound.conf
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

Then restart the server so that it takes the configuration changes:

```
$ sudo unbound-control reload
```

# Test your new validating resolver

Run the following commands and confirm if you receive the "ad" flag:

1. dig @localhost com. SOA 
2. dig @localhost com. SOA +dnssec

Now try the same tests as in Lab DNSSEC 01

1. dig @9.9.9.9   www.dnssec-failed.org +dnssec
1. dig @9.9.9.9   www.dnssec-failed.org +dnssec +cd 
1. dig @localhost www.dnssec-failed.org +dnssec
1. dig @localhost www.dnssec-failed.org +dnssec +cd

What do you notice ? Why ?

# Why does this work?

Please discuss with your peers and/or your instructor

1. How does validation work?
1. Why did it work for your resolver?
1. Where is the trust-anchor?
1. Where did the trust-anchor come from?
1. Check the trust-anchor! Is it correct?

# Temporary desable DNSSEC validation for broken domains

A zone can become broken due to issues with its DNSSEC configuration. In that case, validating resolvers may return DNS answers with status bogus, servfail, etc. Users behind such recursive resolver will get impacted for those domains. While it is generally the responsibility of the domain administrator to fix the issue, the recursive resolver administrator can take action to temporarily disable DNSSEC validation for such domain that is broken at validation level.

Please follow these steps:
1. Run `dig @localhost www.dnssec-failed.org`
1. Run `sudo unbound-control insecure_add dnssec-failed.org`
1. Wait for the cache to expire (up to 5 minutes)
1. Run `dig @localhost www.dnssec-failed.org`
1. Run `sudo unbound-control insecure_remove dnssec-failed.org`
1. Wait for the cache to expire (up to 5 minutes)
1. Run `dig @localhost www.dnssec-failed.org`
