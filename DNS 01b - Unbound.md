# Setting up a recursive server with Unbound

This lab should be executed on your machine resolv1 or resolv2.

We will test resolving first.
```
ping icann.org
```
Let's look at `/etc/resolv.conf`.
```
$ cat /etc/resolv.conf
search grpX.<lab domain>
nameserver 100.100.X.67
nameserver 100.100.X.68
nameserver fd89:59e0:X:64::67
nameserver fd89:59e0:X:64::68
```
So again the resolv1 and resolv2 servers are used for resolving.
Unfortunately we have not yet installed the resolvers.

# Installing software

In this lab we use Unbound, an open-source software developed and maintained by 
the non-profit organisation called NL-netlabs. If you use Unbound commercially, 
please consider contributing.
```
$ sudo apt install unbound
```
Most likely that command did fail. It did so because resolving doesn't work.
We are in a catch22 situation. Luckily for us, the internet today provides
a numbers of public resolvers that we can temporarily use.

Change your `/etc/resolv.conf` file to:
```
nameserver 9.9.9.9
```
Quad9 is a non-profit organisation in Switzerland that provides a free
public resolver service. Check them out at https://quad9.org

Now the installation should work
```
$ sudo apt install unbound
```

# Configuration 

Unbounds configuration is stored in `/etc/unbound/unbound.conf`.
Lets edit the file
```
$ sudo nano /etc/unbound/unbound.conf
```

We will add the options on which interface and port 
to listen for queries, and some other parameters. 
The file should look as follows:
```
server:
        interface: 0.0.0.0
        interface: ::0

        access-control: 0.0.0.0/0 allow
        access-control: ::/32 allow

        port: 53

        do-udp: yes
        do-tcp: yes
        do-ip4: yes
        do-ip6: yes

include: "/etc/unbound/unbound.conf.d/*.conf"
```

Then verify configuration file syntax :

```
# unbound-checkconf
```

If all looks good, you should get something similar to the following:

```
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

If you get something like 
```
/var/lib/unbound/root.key: No such file or directory
[1742580246] unbound-checkconf[1055:0] fatal error: auto-trust-anchor-file: "/var/lib/unbound/root.key" does not exist
```
we must fix this error first.
```
$ sudo apt install dns-root-data
$ sudo cp /usr/share/dns/root.key /var/lib/unbound/root.key
```
Now we should get no errors in the configuration.

Then restart the server so that it takes the configuration changes:
```
# unbound-checkconf
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```
Now we must enable and start the server
```
$ sudo systemctl enable unbound
$ sudo systemctl start  unbound
```

Check the status of the Unbound process:

```
$ sudo systemctl status unbound
```

You should obtain an output similar to the following:

```
● unbound.service - Unbound DNS server     
Loaded: loaded (/lib/systemd/system/unbound.service; enabled; vendor preset: enabled)    
Drop-In: /etc/systemd/system/service.d
	└─lxc.conf     Active: active (running) since Thu 2021-05-13 03:49:11 UTC; 13s ago       Docs: man:unbound(8)
	Process: 571 ExecStartPre=/usr/lib/unbound/package-helper chroot_setup (code=exited, status=0/SUCCESS)
	Process: 574 ExecStartPre=/usr/lib/unbound/package-helper root_trust_anchor_update (code=exited, status=0/SUCCESS)   Main PID: 578 (unbound)      Tasks: 1 (limit: 152822)     Memory: 7.8M     
	CGroup: /system.slice/unbound.service             		└─578 /usr/sbin/unbound -d
May 13 03:49:10 resolv2.grpX.<lab domain> unbound[178]: [178:0] info: [25%]=0 median[50%]=0 [75%]=0
May 13 03:49:10 resolv2.grpX.<lab domain> unbound[178]: [178:0] info: lower(secs) upper(secs) recursions
May 13 03:49:10 resolv2.grpX.<lab domain> unbound[178]: [178:0] info:    0.000000    0.000001 1
May 13 03:49:11 resolv2.grpX.<lab domain> package-helper[577]: /var/lib/unbound/root.key has content
May 13 03:49:11 resolv2.grpX.<lab domain> package-helper[577]: success: the anchor is ok
May 13 03:49:11 resolv2.grpX.<lab domain> unbound[578]: [578:0] notice: init module 0: subnet
May 13 03:49:11 resolv2.grpX.<lab domain> unbound[578]: [578:0] notice: init module 1: validator
May 13 03:49:11 resolv2.grpX.<lab domain> unbound[578]: [578:0] notice: init module 2: iterator
May 13 03:49:11 resolv2.grpX.<lab domain> unbound[578]: [578:0] info: start of service (unbound 1.9.4).
May 13 03:49:11 resolv2.grpX.<lab domain> systemd[1]: Started Unbound DNS server.
```

# Test your new resolver

Run the following commands and see if you receive answers:

1. dig @localhost    com. SOA +noall +answer
1. dig @100.100.X.67 com. SOA +noall +answer
1. dig @100.100.X.68 com. SOA +noall +answer

# Restore resolv.conf

Restore `/etc/resolv.conf` to its original content:
```
search grpX.<lab domain>
nameserver 100.100.X.67
nameserver 100.100.X.68
nameserver fd89:59e0:X:64::67
nameserver fd89:59e0:X:64::68
```

# Test your resolver again

You should get the same answers as above

1. dig com. SOA +noall +answer
1. dig com. SOA +noall +answer
1. dig com. SOA +noall +answer
