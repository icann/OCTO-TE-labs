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

# TSIG

From [RFC 8945](https://www.rfc-editor.org/rfc/rfc8945):

        This document specifies use of a message authentication code [...] to provide an 
        efficient means of point-to-point authentication and integrity checking for DNS 
        transactions.

The Transaction Signatures (TSIG) can be used to authenticate DNS requests and responses.
This applies for example to Dynamic DNS (DDNS) updates or zone transfers.

In this lab we will be using TSIG authentication instead of IP addresses to authenticate zone 
transfer.

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

## Adding keys on your Primary

Add the tsig key at the bottom of **named.conf.options** config file.

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

Don't forget to replace X in "**grpX-key**" ....!

Then in your zone, change allow-transfer line

```
zone "grpX.lab_domain.te-labs.training" {                                                                               
        [...]
        allow-transfer { key grpX-key; };
        [...]
};
```

As you can see above, we've changed “allow-transfer” statement allowing transfer of the zone for holders of the "tsig-key".

Restart *named* service

```
$ named-checkconf
$ rndc reconfig
```

## Check if it works

Test that zone transfer has stopped working.
```
$ dig @100.100.X.66 axfr grpX.lab_domain.te-labs.training

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
$ dig @100.100.X.66 -y hmac-sha256:grpX-key:THIS_IS_MY_KEY axfr grpX.lab_domain.te-labs.training
```


### Add the TSIG key to your NS1 configuration

In **/etc/bind/named.conf.options**, add the tsig key, and a statement to tell which key to use when talking to “100.100.X.66;” (the soa server ):

```
key "grpX-key" {
        algorithm hmac-sha256;
        secret "THIS_IS_MY_KEY";
};

server 100.100.X.66 {		// here you put the IP of YOUR primary server (SOA)
        keys { grpX-key; };
};
server fd89:59e0:X:64::66 {		// here you put the IP of YOUR primary server (SOA)
        keys { grpX-key; };
};
```

Save, exit and restart bind9.

### Testing the configuration

On SOA server increase the serial and reload the zone. Then, 
`$ sudo rndc reload grpX.lab_domain.te-labs.training`

In ns1, go to logs and validate that the transfer was successful.

```
$ tail /var/log/syslog

zone grp2.lab_domain.te-labs.training/IN: Transfer started.
transfer of 'grp2.lab_domain.te-labs.training/IN' from 100.100.2.66#53: connected using 100.100.2.13>
zone grp2.lab_domain.te-labs.training/IN: transferred serial 2022052401: TSIG 'grp2-key'
transfer of 'grp2.lab_domain.te-labs.training/IN' from 100.100.2.66#53: Transfer status: success
transfer of 'grp2.lab_domain.te-labs.training/IN' from 100.100.2.66#53: Transfer completed: 1 messag>
zone grp2.lab_domain.te-labs.training/IN: sending notifies (serial 2022052401)
managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer complete)
resolver priming query complete
```

## On NS2 server

Edit **/etc/nsd/nsd.conf** file, create key section and add the tsig key grpX-key

```
key:
        name: "grpX-key"
        algorithm: hmac-sha256
        secret: "THIS_IS_MY_KEY"
```
and Fix pattern:

change these lines

```
allow-notify: 100.100.X.66 NOKEY
request-xfr: AXFR 100.100.X.66 NOKEY
```
with this:

```
allow-notify: 100.100.X.66 grpX-key
request-xfr: AXFR 100.100.X.66 grpX-key
```


Save, exit, verify and restart NSD service.

```
$ nsd-checkconf /etc/nsd/nsd.conf
$ sudo nsd-control reconfig
$ sudo nsd-control reload grpX.lab_domain.te-labs.training
```

Check the logs on NS2 and on SOA.
