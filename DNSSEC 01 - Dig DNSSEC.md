# Dig DNSSEC

> [!TIP]
> Please finish the lab [DNS 02a - Dig](DNS%2002a%20-%20Dig.md) first, if you haven't done so already.

## Using dig to get DNSSEC information

* Getting the resource record signatures (RRSIG) for signed domains: 

```
$ dig www.icann.org A +dnssec
$ dig icann.org NS +dnssec
```

Compare the number of RRSIG you get in the first case to the number you received in the second case.

You can add the `+multi` option to make the results more "readable".

* Retrieve the public keys for the zone: they are stored in a specific resource record type named "DNSKEY"

```
$ dig icann.org DNSKEY +noall +answer
$ dig icann.org DNSKEY +noall +answer +multi
```

You can mix the known options such as redirecting to a specific name server, adding multiline option, etc.

* Retrieve the delegation signer info for the zone: they are stored in a specific resource record type named "DS"

```
$ dig icann.org DS
```

There are also some special DNSSEC records that you can not actually send queries for,
but will be contained in negative answers. Look for NSEC or NSEC3 records.

```
$ dig sdlkhghkj.icann.org +dnssec
```

# Checking Disabled 

1. dig @9.9.9.9      www.dnssec-failed.org +dnssec
1. dig @9.9.9.9      www.dnssec-failed.org +dnssec +cd 
1. dig @100.100.X.68 www.dnssec-failed.org +dnssec
1. dig @100.100.X.68 www.dnssec-failed.org +dnssec +cd
