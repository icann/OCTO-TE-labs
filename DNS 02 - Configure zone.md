# Configure primary authoritative server

We are going to configure a hidden authoritative server and create the authoritative zone **grpX.\<lab domain\>**.

> [!TIP]
> This is a very important DNS concept! 
> If you haven't heard of or unsure about the "hidden primary" 
> configuration, please ask your instructor. 

## From the parent zone

Our "parent" (**\<lab domain\>**) has already created the following in its own zone:

```
; grpX
grpX             NS          <lab domain>.
```

The lab uses dnsdist (a DNS proxy) to forward queries to the 
respective groups ns1 and ns2 servers. It expects these server at the
following ip addresses:

| Device Name   | IPv4 Address   | IPv6 Address         | 
| ------------- | -------------- | -------------------- |
| ns1           | 100.100.X.130  | fd73:7c99:X:128::130 |
| ns2           | 100.100.X.131  | fd73:7c99:X:128::131 |

Our zone configuration must be compatible with that.

## Setting up the authoritative zone

Use the "SOA" server as primary authoritative server for the  **grpX.\<lab domain\>** zone.

Please follow [DNS 02a](http://DNS%2002a%20-%20Bind.md) for installation of the primary server.

Your instructor will tell you which instructions to follow for installation of your secondary servers.

- [Bind9](http://DNS%2002b%20-%20Bind%20Secondary.md)
- [Knot DNS](http://DNS%2002c%20-%20Knot)
- [NSD](http://DNS%2002d%20-%20NSD)
- [PowerDNS](http://DNS%2002e%20-%20PowerDNS)

Once you are done with the configuration of your primary and secondary servers, please come back here and conitinue with the next section!

## Test your zone configuration and propagation.

We will now use *dig* tool to verify the zone configuration and propagation, then do the same for one or two other groups in the class and share comments. From your client, run the following dig queries. All should return **answer section** otherwise you must review your configurations before continuing:

On the **cli** instance

1. `dig @100.100.X.66  grpX.<lab domain>. SOA`
1. `dig @100.100.X.130 grpX.<lab domain>. SOA`
1. `dig @100.100.X.131 grpX.<lab domain>. SOA`
1. `dig @100.100.X.131 grpX.<lab domain>. SOA +short`
1. `dig @100.100.X.131 grpX.<lab domain>. SOA +multi`
1. `dig @100.100.X.130 grpX.<lab domain>. NS  +multi`
1. `dig @100.100.X.131 grpX.<lab domain>. NS`
1. `dig grpX.<lab domain>. NS +nsid`

Please repeat the following queries at least once
```
1. `dig @<lab domain>. NS +nsid`
1. `dig @<lab domain> hostname.bind txt CHAOS`
1. `dig @<lab domain> version.bind  txt CHAOS`
1. `dig @<lab domain> id.server     txt CHAOS`
```
