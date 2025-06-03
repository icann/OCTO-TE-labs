# Set up a BIND validating recursive server.

Follow this lab on the same machine you installed the bind resolver in Lab DNS 01.

Our current configuration in `/etc/bind/named.conf.options` should be
```
options {
  directory "/var/cache/bind";
  dnssec-validation no;
  listen-on port 53 { localhost; 100.100.0.0/16; };
  listen-on-v6 port 53 { localhost; fd89:59e0::/32; };
  allow-query { any; };
  recursion yes;
};
```

To do this, edit the file :

```
$ sudo nano named.conf.options
```

You might already have spotted the culprit, please change `dnssec-validation no;` to `dnssec-validation auto;`

Save your changes and reload the configuration

```
$ rndc reload
```

# Test your new validating resolver

Run the following commands and confirm if you receive the "ad" flag:

1. dig @localhost com SOA 
2. dig @localhost com SOA +dnssec

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

# Temporary disable DNSSEC validation for broken domains

A zone can become broken due to issues with its DNSSEC configuration. In that case, validating resolvers may return DNS answers with status bogus, servfail, etc. Users behind such recursive resolver will get impacted for those domains. While it is generally the responsibility of the domain administrator to fix the issue, the recursive resolver administrator can take action to temporarily disable DNSSEC validation for such domain that is broken at validation level.

Please follow these steps:
1. Run `dig @localhost www.dnssec-failed.org`
1. Run `rndc nta dnssec-failed.org`
1. Run `dig @localhost www.dnssec-failed.org`
1. Run `rndc nta -remove dnssec-failed.org`
1. Run `dig @localhost www.dnssec-failed.org`

Please discuss with your peers and your instructor:
1. What happend?
1. Why did the second dig command succeed?
2. Why did the third fail again?

Please checkout the help page for rndc. You can even specifiy a lifetime for negative trust-anchors.


