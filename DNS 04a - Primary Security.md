# Primary Security

Hopefully your servers are configured in a hidden primary configuration.
That has the great benefit, that your primary server doesn't have to be available on the internet, 
at least not for most of the internet.

# Hardening your OS

This something we will not do in this lab, but something you **MUST** do
in your production environment. Restrict access to the machine, close all ports, no
access from the internet, ...

# Hardening DNS

Most secure would be to remove your primary from the internet completly. But that would also 
make it totaly useless. Therefore we need to check what our server needs to do and how it 
needs to communicate.

Must allow:
- send notify to secondaries 
- allow secondaries to request axfr/ixfr

Must not allow:
- queries from the internet
- axfr/ixfr requests from other servers then secondaries
- recursion

Depending on how you update your zone, you might want to allow DDNS updates from some specific
machines, but not from the internet. 

# Recursion

First test your server
```
$ dig @100.100.X.66 icann.org
```
If the status in the response is not `REFUSED` or you even got an ip address, your server allows recursion.
The good new is, this is easy to fix.

Take a look at your `named.conf.options` file. What would you change?

Test your changes!

# TSIG

From [RFC 8945](https://www.rfc-editor.org/rfc/rfc8945):

        This document specifies use of a message authentication code [...] to provide an 
        efficient means of point-to-point authentication and integrity checking for DNS 
        transactions.

The Transaction Signatures (TSIG) can be used to authenticate DNS requests and responses.
This applies for example to Dynamic DNS (DDNS) updates or zone transfers.

In this lab we will be using TSIG authentication instead of IP addresses to authenticate zone 
transfer.

Let's start by testing if our server allows zone transfers or if we are already secure
```
$ dig @100.100.X.66 axfr grpX.lab_domain

## Generating keys

On your primary server (SOA):

Generate the tsig key 

```
$ sudo tsig-keygen -a hmac-sha256 grpX-key > /tmp/grpX-key.txt
```

Check the content of the file. Should look similar to this:

```
key "grpX-key" {
	algorithm hmac-sha256;
	secret "THIS_IS_MY_KEY";
}; 
```

## Adding keys

Add the tsig key at the bottom of **named.conf.options** config file.

Then in your zone, change allow-transfer line

```
zone "grpX.lab_domain" {                                                                               
        [...]
        allow-transfer { key grpX-key; };
        [...]
};
```
Check the configuration and reconfigure the *named* service

```
$ named-checkconf
$ rndc reconfig
```

## Check if it works

Test that zone transfer has stopped working.
```
$ dig @100.100.X.66 axfr grpX.lab_domain

...
; Transfer failed.
```

A look into the SOA server logs should show something like:

```
$ tail /var/log/syslog

24-May-2022 10:03:29.433 client @0x7f185c006920 100.100.X.130#38993 (grpX.lab_domain): zone transfer 'grpX.lab_domain/AXFR/IN' denied
```

We need the key!

You can also test manually as follows:

```
$ dig @100.100.X.66 -y hmac-sha256:grpX-key:THIS_IS_MY_KEY grpX.lab_domain AXFR
```

 

 - PUT KEY ON SECONDARIES
 - NOTIFY WITH KEY
 







```
key "grpX-key" {
algorithm hmac-sha256;
        secret "THIS_IS_MY_KEY";
};
server 100.100.X.130 {
     keys {grpX-key ; };
};
server 100.100.X.131 {
     keys {grpX-key ; };
};
server fd89:59e0:X:128::130 {
     keys {grpX-key ; };
};
server fd89:59e0:X:128::131 {
     keys {grpX-key ; };
};
```