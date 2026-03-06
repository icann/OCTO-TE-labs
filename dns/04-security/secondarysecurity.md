---
layout: page
title: "Secondary Security"
parent: "Security"
nav_order: 3
---

<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

# Lab Hardening your DNS config (Intro)

```
Created by: Yazid AKANHO
version: 2024042400
Previous revision : 2024030100
Modified by: -
```


Now that your zone configuration is working fine (if not, please do not continue here and go back to fix the config before you continue), we need to do some security enhancements to protect our DNS against some types of attacks.

##Open Zone transfer
Try to do a manual zone transfer using respectively ns1, ns2 and SOA server IP address from our client machine. What do you notice ? 

Tip: the command to use is ***dig axfr <*domain*> @nameserver***. 


Do some Internet research and talk with your classmates to find a solution to avoid open zone transfer.

####Fix the open zone transfer
If your answer to the above question is ***allow-transfer***, then you are right.
Go ahead and apply the appropriate configuration update to fix the issue discussed above.
Then, test again the zone transfer. Is the issue resolved ?
> Note: allow-transfer is not the single way to achieve this. Other methods exist.


## Hidden primary
Is your hidden primary server really "hidden" ? Try to query your zone on it from any other VM and confirm wheather it replies or not. After this test, can you confirm if it is really "hidden" ?

Discuss with other participants to find a solution to make it a real "hidden" primary nameserver; i.e. it should not respond to DNS queries as it is the primary and not supposed to serve the zone publicly. This is a best practice in DNS. However, do not forget that for troubleshooting purpose, you may always need to query it from some specific (or known) IP addresses.

Apply the solution after you have agreed on a consensus with the training facilitators.

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
server fd89:59e0:X:64::66 {	// here you put the IP of YOUR primary server (SOA)
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
