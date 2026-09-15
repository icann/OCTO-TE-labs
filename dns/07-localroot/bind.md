---
layout: page
title: "Bind"
parent: "Local Root"
nav_order: 1
---

# Local Root with Bind

BIND has built-in support for maintaining a local mirror of the DNS root zone.

For the root zone, this is implemented using a zone of type:

```text
mirror
```

BIND has a built-in list of root-zone primary servers, so the basic configuration is very small. Current BIND documentation specifically recommends `type mirror` for a local copy of the root zone.

# Check the current configuration

Before changing anything, find the current root-zone configuration:

```bash
grep -R -n 'zone "."' /etc/bind/named.conf*
```

On a typical installation you will find a root hint zone similar to:

```text
zone "." {
    type hint;
    file "/usr/share/dns/root.hints";
};
```

The exact filename may differ on your system.

This configuration tells BIND to use root hints to find the remote root servers.

# Replace the root hint zone

We want BIND to maintain a local mirror instead.

Edit the configuration file containing the existing `zone "."` declaration.

Change:

```text
zone "." {
    type hint;
    file "/usr/share/dns/root.hints";
};
```

to:

```text
zone "." {
    type mirror;
    file "/var/cache/bind/root.zone";
};
```

Do not leave two definitions for:

```text
zone "."
```

There must be only one active root-zone definition for the resolver.

The `file` option tells BIND where to store the current mirrored root zone so that the data can survive a restart. BIND documents persistent storage for mirror zones through this option.

# Check the configuration

Run:

```bash
named-checkconf
```

There should be no errors.

If you see an error saying that the root zone has already been configured, find the other:

```text
zone "."
```

definition and remove or replace it.

# Restart BIND

Restart the resolver:

```bash
systemctl restart bind9
```

Check that it is running:

```bash
systemctl status bind9
```

If BIND did not start, inspect the log:

```bash
journalctl -u bind9 --no-pager -n 50
```

# Check the mirror zone

Ask BIND for the status of the root zone:

```bash
rndc zonestatus .
```

You should see information about the root zone, including its current serial number.

BIND's `zonestatus` command reports the current serial number and refresh/expiry information for the zone.

Also check that the mirror file exists:

```bash
ls -lh /var/cache/bind/root.zone
```

The file should contain a complete root-zone copy.

# Query the local root

From the resolver itself:

```bash
dig @127.0.0.1 . SOA
```

and:

```bash
dig @127.0.0.1 . NS
```

You should receive the root-zone SOA and NS records.

Now ask for a TLD delegation:

```bash
dig @127.0.0.1 com NS
```

The resolver should return the `.com` name servers.

Finally test a normal query:

```bash
dig @127.0.0.1 icann.org A
```

The query should resolve normally.

# Verify that the root zone is DNSSEC validated

Mirror zones are special in BIND because the entire zone is DNSSEC validated before a new version is used.

If a new version cannot be validated, BIND keeps using the last correctly validated version while it attempts to obtain a valid copy.

Check the root DNSKEY:

```bash
dig @127.0.0.1 . DNSKEY +dnssec
```

You should receive the root DNSKEY records.

Check the resolver's DNSSEC validation status:

```bash
rndc validation status
```

# Compare the local root with a remote root

Ask your local resolver for the root SOA:

```bash
dig @127.0.0.1 . SOA
```

Now ask a remote root server:

```bash
dig @198.41.0.4 . SOA
```

Compare the SOA serial numbers.

They should refer to the same root-zone version.

You have now demonstrated that BIND is maintaining a local copy of the actual root zone rather than using a manually created approximation.

# Observe the root-zone transfer

If you want to see the maintenance mechanism, increase the BIND logging level temporarily or watch the system journal while forcing a retransfer.

First:

```bash
rndc retransfer .
```

Then watch:

```bash
journalctl -u bind9 -f
```

The exact log messages depend on the BIND version and configuration.

The important point is that BIND obtains the root zone from the configured root-zone primary service and validates it before using it.

# Test persistence

The purpose of the `file` setting is to retain the mirrored zone between restarts.

Check the file:

```bash
ls -lh /var/cache/bind/root.zone
```

Then restart BIND:

```bash
systemctl restart bind9
```

After the restart:

```bash
rndc zonestatus .
dig @127.0.0.1 . SOA
```

The root zone should be available again.

# Understand the fallback behavior

A mirror zone is not simply a permanently static copy.

If BIND has no usable mirrored root-zone data, it can fall back to normal recursive behavior. Current BIND documentation describes this as using traditional DNS recursion when no usable mirror-zone data is available.

This is useful operationally, but it is important to understand what it means for the lab:

> A successful DNS lookup does not, by itself, prove that the query was answered from the local mirror.

The mirror is a local copy of the root zone that BIND prefers when it is available.

# Optional experiment: observe root traffic

You can investigate the difference using packet capture.

Run:

```bash
tcpdump -n host 198.41.0.4 and port 53
```

Then generate a query that requires a new resolution path.

Remember that cached data may prevent any root query from occurring.

To make the observation more useful, clear the relevant BIND cache before testing:

```bash
rndc flush
```

Then repeat the query.

The important lesson is that caching and local-root mirroring are different mechanisms.

# Troubleshooting

If the mirror does not become active, check:

```bash
named-checkconf
rndc zonestatus .
journalctl -u bind9 --no-pager -n 100
```

If the zone is not updating, check that the resolver can reach the root-zone transfer service over the network.

If DNSSEC validation fails, do not disable validation just to make the lab work. The root-zone mirror is supposed to be validated.

# Final configuration

The essential configuration is:

```text
zone "." {
    type mirror;
    file "/var/cache/bind/root.zone";
};
```

That is all that is required for the root zone when using a current BIND version with the built-in root-server primaries.

# What you have learned

With this configuration BIND:

1. Maintains a local copy of the root zone.
2. Updates that copy automatically.
3. Validates the complete mirrored zone with DNSSEC.
4. Uses the local root data for recursive resolution.
5. Retains the copy on disk across restarts.

The local root is therefore a component of the recursive resolver, not a public authoritative root service.
