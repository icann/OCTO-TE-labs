---
layout: page
title: "Primary Bind"
parent: "Security"
nav_order: 2
---

# Primary Security

Hopefully your servers are configured in a hidden primary configuration.
That has the great benefit, that your primary server doesn't have to be available on the internet, at least not for most of the internet.

# Hardening your OS

This something we will not do in this lab, but something you **MUST** do
in your production environment. Restrict access to the machine, close all ports, no
access from the internet, ...

# Hardening DNS

Most secure would be to remove your primary from the internet completly. But that would also 
make it totaly useless. Therefore we need to check what our server needs to do and how it 
needs to communicate.

## Hidden primary
Is your hidden primary server really "hidden" ? Try to query your zone on it from any other VM and confirm wheather it replies or not. After this test, can you confirm if it is really "hidden" ?

Discuss with other participants to find a solution to make it a real "hidden" primary nameserver; i.e. it should not respond to DNS queries as it is the primary and not supposed to serve the zone publicly. This is a best practice in DNS. However, do not forget that for troubleshooting purpose, you may always need to query it from some specific (or known) IP addresses.

Apply the solution after you have agreed on a consensus with the training facilitators.

### To-Do List

Must allow:
- allow secondaries to request axfr/ixfr

Must not allow:
- recursion
- axfr/ixfr requests from other servers then secondaries
- queries from the internet

Depending on how you update your zone, you might want to allow DDNS updates from some specific
machines, but not from the internet. (Out-of-scope for this lab)

# Recursion

First test your server
```
$ dig @100.100.X.66 icann.org
```
If the status in the response is not `REFUSED` or you even got an ip address, 
your server allows recursion. The good new is, this is easy to fix.

Take a look at your `named.conf.options` file. What would you change?

Test your changes!

# AXFR / IXFR

This topic is a bit more tricky. We can't just disable it all together,
but we must allow some well-known servers to request AXFR/IXFR, but block the 
rest of the world from doing it.

Work with your peers and try to request and AXFR from their soa server.

## TSIG

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
dig @100.100.X.66 grpX.lab_domain AXFR
```

## Generating TSIG keys

Generate the tsig key on your primary server (SOA).
```
sudo tsig-keygen -a hmac-sha256 grpX-key
```
Add the output to your named.conf.options file. Should look similar to this:

```
key "grpX-key" {
	algorithm hmac-sha256;
	secret "THIS_IS_MY_KEY";
}; 
```

## Adding TSIG keys

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
named-checkconf
rndc reconfig
```
Check if you can do a zone transfer
```
dig @100.100.X.66 grpX.lab_domain AXFR
```
This should fail. Try again, but this time specify the TSIG-key
```
dig @100.100.X.66 grpX.lab_domain AXFR -y hmac-sha256:grpX-key:THIS_IS_MY_KEY
```

## Secure Notify

The primary server sends notifies to the secondaries.

Please include the following statements in `named.conf.options`
```
server 100.100.X.130 {
     keys { grpX-key; };
};
server 100.100.X.131 {
     keys { grpX-key; };
};
server fd89:59e0:X:128::130 {
     keys { grpX-key; };
};
server fd89:59e0:X:128::131 {
     keys { grpX-key; };
};
```

# Queries from the Internet

Work together with one or more other groups and see if they can query 
your soa server and if you can query theirs.

The goal is to stop all your peers from being able to query your soa server,
but still you should be able to query it yourself.

Look at your server configuration. What would you change?

# Zone Update

Please edit the zone file of your domain `grpx.lab_domain`. Increase the serial 
number, save the file and reload the zone `sudo rndc reload grpX.Lab_domain`.

Check if your primary and your secondary serve the same zone version.
```
dig @100.100.X.66  grpX.lab_domain SOA +noall +nocomments +answer
dig @100.100.X.130 grpX.lab_domain SOA +noall +nocomments +answer
dig @100.100.X.131 grpX.lab_domain SOA +noall +nocomments +answer
```

Why did you get these answer?

Please continue with the labs for secondary security.
