# Introduction to Domain Information Groper (dig)

Working with "dig" and understanding its outputs are crucial for DNS troubleshooting and debugging, so don't be shy and ask questions to learn the maximum. 

> [!TIP]
> We can't explore all dig possibilities during this workshop. We do recommend you to continue playing to learn this fantastic tool.

## Goals

1. Become familiar with the use of dig
2. Use dig to retrieve and examine DNS ressource records
3. Interpret dig's output and identify some common DNS problems.

## The dig Tool

The tool “*dig*” was originally shipped with BIND and is commonly found on many Unix-like platforms. It stands for *Domain Information Groper*. It collects data about Domain Name Servers and is helpful for troubleshooting DNS problems and/or displaying DNS information.

Other DNS implementations also include similar tools, often with similar names (e.g. kdig). Older tools used for DNS troubleshooting include *nslookup* and *host*.

> [!WARNING]
> Do not use these older tools, they work very poorly with modern DNS features.

A manual page for dig can be found here or from the command-line. There are a lot of available parameters. You can ignore most of them while you are getting started.

```
$ man dig
```

A typical invocation of dig looks like:

```
dig @SERVER NAME TYPE
```

- **SERVER**: name or IP address of the name server to query. If no server argument is provided, dig consults /etc/resolv.conf. If no usable addresses are found, dig will send the query to the local host.
- **NAME**: name of the resource record that is to be looked up.
- **TYPE**: type of query requested (A, MX, NS, SIG, ...). If no type supplied, dig will lookup for an A record.

For each dig query sent, a response is expected with different sections. Here are few of them:

* The first line displays the version of the dig command.
* The **HEADER** section shows the information it received from the server. **Flags** refer to the answer format and they are extremely important to understand the overall result of the query. There are six (06) flags: 

	1. **AA**: Authoritative Answer
	2. **TC**: Truncated Response
	3. **RD**: Recursion Desired
	4. **RA**: Recursion Available
	5. **AD**: Authentic Data
	6. **CD**: Checking Disabled
* The **OPT PSEUDOSECTION** displays advanced data such as EDNS (Extension mechanisms for DNS), if used.
* The **QUESTION** section displays the query data that was sent.
* The **ANSWER** section: probably the most important section for the user.
* The **ADDITIONAL** section: contains data that might be usefull in the further processing of the answer
* The **STATISTICS** section shows metadata about the query: 
   * query time (amount of time to get the response); 
   * SERVER (IP address and port of the responding DNS server); 
   * WHEN (timestamp when the command was run); 
   * MSG SIZE rcvd (size of the response packet from the DNS server).

## Sending DNS Queries Using dig

Try using dig to query the IPv4 address corresponding to www.icann.org. Here are various ways of doing that; what differences do you see in the output from each of them?

```
$ dig www.icann.org A
$ dig @8.8.8.8 www.icann.org A
$ dig @one.one.one.one www.icann.org A
```

For each answer that you got, discuss the different section of the answer with the facilitators. You will be surprised to see that dig tool provides a sea of information.

Now try other records

```
$ dig icann.org NS
$ dig ricta.org.rw SOA
$ dig www.ricta.org.rw A
$ dig @8.8.8.8 www.ricta.org.rw A
$ dig @ns1.ricta.org.rw ricta.org.rw SOA
```

Again, try to discuss the various outputs with your instructors.

## Let's continue exploring dig tool with its following options

* **+short**: to display only the queried resource record value

```
$ dig www.icann.org A +short
```

* **+noall +answer**: to get detailed information of the answers section only

```
$ dig www.icann.org A +noall +answer
```

* **+trace**: lists each different server the query goes through to its final destination. Good for troubleshooting

```
$ dig www.icann.org A +trace
```

* **-x**: to lookup a domain name by its IP address (**reverse lookup**)

```
$ dig -x 192.0.47.7
```

* **-f**: to look up multiple entries stored in a file

```
$ echo "icann.org google.com gmail.com" > test_batch_lookup.txt ; dig -f test_batch_lookup.txt +noall +answer
```

# Server Identity

Back in the days engineers thought it was a good idea that you could query
a server for the software and version it is running. Turns out, bad guys can 
exploit that information. So today most servers don't answer anything or
some preconfigured string, but not the actuall software or version that is running.

BIND servers respond to queries for name version.bind with record type TXT and class CHAOS. By default, this is set to the version of BIND that has been installed


The first instances of this have been made by bind, which is reflected in the 
query name.

* **hostname.bind**: to retrieve the hostname of the server (if allowed to)
```
$ dig @ns.icann.org. hostname.bind TXT CHAOS
$ dig @d.root-servers.net hostname.bind TXT CHAOS
$ dig hostname.bind TXT CHAOS
```

* **version.bind**: to retrieve the version of the server (if allowed to)
```
$ dig @ns.icann.org. version.bind TXT CHAOS
$ dig @d.root-servers.net version.bind TXT CHAOS
$ dig version.bind TXT CHAOS
```

A newer approch has been the query for **id.server** as it is independent of the 
software manufacturer.
* **id.server**: 
```
$ dig @ns.icann.org. id.server TXT CHAOS
$ dig @d.root-servers.net id.server TXT CHAOS
$ dig id.server TXT CHAOS
```

All these old query types have one problem. They are a query in itself.
In modern anycast networks we can't be sure that two queries to the same IP address
will be received by the same server. So the latest approch allows for the
server identity to be included in the response.

* **nsid**: retrieve DNS Name Server Identifier (NSID) 
```
$ dig @ns.icann.org. icann.org SOA +nsid
$ dig @d.root-servers.net icann.org SOA +nsid
$ dig icann.org SOA +nsid
```

### Using dig to get DNSSEC information

* Getting the resource record signatures (RRSIG) for signed domains: 

```
$ dig www.icann.org A +dnssec
$ dig icann.org NS +dnssec
```

Compare the number of RRSIG you get in the first case to the number you received in the second case.

You can add the `+multi` option to make the results more "readable".

* Retrieve the public keys for the zone: they are stored in a specific resource record type named "DNSKEY"

```
$ dig icann.org DNSKEY +multi
```

You can mix the known options such as redirecting to a specific name server, adding multilign option, etc.

* Retrieve the delegation signer info for the zone: they are stored in a specific resource record type named "DS"

```
$ dig icann.org DS
```

There are also some special DNSSEC records that you can not actually send queries for,
but will be contained in negative answers. Look for NSEC or NSEC3 records.

```
$ dig sdlkhghkj.icann.org +dnssec
```
