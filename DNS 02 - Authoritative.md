# Configure primary authoritative server

We are going to build a "hidden primary" setup, where the SOA server is the hidden primary and NS1 and NS2
will server the zone externaly.

> [!IMPORTANT]
> "Hidden primary" is a very important DNS concept! 
> If you haven't heard of or unsure about the 
> configuration, please ask your instructor. 

## From the parent zone

Our "parent" (***\<lab domain\>***) has already created the following in its own zone:

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

## Setting up the primary

Use the "SOA" server as primary authoritative server for the  **grpX.\<lab domain\>** zone.

Your instructor will tell you which instructions to follow for installation of your primary server.

- [DNS 02a - Bind](http://DNS%2002a%20-%20Primary%20Bind.md) 

## Setting up the secondaries

Your instructor will tell you which instructions to follow for installation of your secondary servers.

- [Bind9](https://github.com/icann/OCTO-TE-labs/blob/extended/DNS%2002b%20-%20Secondary%20Bind.md)
- [NSD](http://DNS%2002b%20-%20Secondary%20NSD)

Once you are done with the configuration of your primary and secondary servers, please come back here and continue with the next section!

## Test your zone configuration and propagation.

We will now use *dig* tool to verify the zone configuration and propagation, then do the same for one or two other groups in the class and share comments. From your client, run the following dig queries. All should return **answer section** otherwise you must review your configurations before continuing:

On the **cli** instance

1. `dig @100.100.X.66  grpX.<lab domain> SOA`
1. `dig @100.100.X.130 grpX.<lab domain> SOA`
1. `dig @100.100.X.131 grpX.<lab domain> SOA`

Please repeat the following queries at least once
```
1. `dig @<lab domain> grpX.<lab domain> NS +nsid`
1. `dig @<lab domain> hostname.bind     TXT CHAOS`
1. `dig @<lab domain> version.bind      TXT CHAOS`
1. `dig @<lab domain> id.server         TXT CHAOS`
```
