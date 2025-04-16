# Setting up a recursive server with Unbound

This lab should be executed on your machine resolv1 or resolv2.

We will test resolving first.
```
ping icann.org
```
Let's look at `/etc/resolv.conf`.
```
$ cat /etc/resolv.conf
search grpX.lab_domain
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
        access-control: ::/0 allow

        port: 53

        do-udp: yes
        do-tcp: yes
        do-ip4: yes
        do-ip6: yes

        trust-anchor-file:
        auto-trust-anchor-file:
        trust-anchor:
        trusted-keys-file:

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
Before we restart unbound let's look at its current status
```
$ sudo unbound-control status
```
Output should be similiar to this
```
version: 1.19.2
verbosity: 1
threads: 1
modules: 3 [ subnetcache validator iterator ]
uptime: 248 seconds
options: reuseport control(namedpipe)
unbound (pid 2233) is running...
```
Now restart the server with the new configuration. Because we made changes to the interface configuration,
we have to do a hard restart, not just a configuration reload.
```
$ sudo systemctl restart unbound
```
Check the status of the Unbound again, look at the uptime counter it should be only a few seconds now.

# Test your new resolver

Run the following commands and see if you receive answers:

1. dig @localhost    com. SOA +noall +answer
1. dig @100.100.X.67 com. SOA +noall +answer
1. dig @100.100.X.68 com. SOA +noall +answer

# Restore resolv.conf

Restore `/etc/resolv.conf` to its original content:
```
search grpX.lab_domain
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
