# Setting up a recursive server with Bind9

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

In this lab we use Bind9. An open-source software developed and maintained by 
the non-profit organisation called Internet Systems Consortium (ISC). If you use 
Bind9 commercially, please consider contributing.
```
$ sudo apt install -y bind9
```
Most likely that command did fail. It did so because resolving doesn't work.
We are in a catch22 situation. Luckily for us, the internet today provides
a numbers of public resolvers that we can temporarily use.

Change your `/etc/resolv.conf` file to:
```
$ sudo mv /etc/resolv.conf /etc/resolv.conf.orig
$ echo "nameserver 9.9.9.9"|sudo tee /etc/resolv.conf
```
Quad9 is a non-profit organisation in Switzerland that provides a free
public resolver service. Check them out at https://quad9.org

Now the installation should work
```
$ sudo apt install -y bind9
```
And we add our own user to the bind group so we can use rndc without sudo.
```
$ sudo adduser sysadm bind
```
> [!TIP]
> Close and reopen your shell window. The new user permissions only get active after logging out and in again.

# Configuration 

Finally we need to start the configuration. Bind9 uses three configuration files
`named.conf`, `named.conf.local` and `named.conf.options`. All are located in `/etc/bind`.

In it's default state bind will not answer queries.

At this point we must configure some BIND9 options.
To do this, edit the file `/etc/bind/named.conf.options`:

```
$ sudo nano named.conf.options
```

Then add the options to indicate (when resolving) the IP addresses 
that are allowed to query this resolver, and the IP addresses our 
server will be listening on (port 53). 

The file should be as follows:

```
options {
  directory "/var/cache/bind";
  dnssec-validation no;
  listen-on port 53 { localhost; 100.100.0.0/16; };
  listen-on-v6 port 53 { localhost; fd89:59e0::/32; };
  allow-query { any; };
  recursion yes;
};
```

Once finish editing the configuration file, verify the configuration syntax:

```
# named-checkconf
```

Then restart the server so that it takes the configuration changes:

```
$ sudo systemctl restart named 
```

Check the status of the bind9 process:

```
# sudo systemctl status named
```

You should get something similar to the below:

```
● named.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/service.d
       └─lxc.conf
   Active: **active (running)** since Thu 2021-05-13 01:38:27 UTC; 4s ago
    Docs: man:named(8)
  Main PID: 849 (named)
   Tasks: 50 (limit: 152822)
   Memory: 103.2M
   CGroup: /system.slice/named.service
       └─849 /usr/sbin/named -f -u bind

May 13 01:38:27 resolv1.grpX.lab_domain named[849]: **command channel listening on ::1#953**
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: managed-keys-zone: loaded serial 6
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: zone 0.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: zone 127.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: zone localhost/IN: loaded serial 2
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: zone 255.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: all zones loaded
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: running
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: managed-keys-zone: Key 20326 for zone . is now trusted [...]
May 13 01:38:27 resolv1.grpX.lab_domain named[849]: resolver priming query complete
```

Alternatively we can check the status of Bind9 with the rndc tool
```
$ rndc status
version: BIND 9.20.9-1+ubuntu24.04.1+deb.sury.org+1-Ubuntu (Stable Release) <id:>
running on localhost: Linux x86_64 6.8.0-1029-aws #31-Ubuntu SMP Wed Apr 23 18:42:41 UTC 2025
boot time: Tue, 27 May 2025 11:15:43 GMT
last configured: Tue, 27 May 2025 11:15:43 GMT
configuration file: /etc/bind/named.conf
CPUs found: 4
worker threads: 4
number of zones: 101 (100 automatic)
debug level: 0
xfers running: 0
xfers deferred: 0
xfers first refresh: 0
soa queries in progress: 0
query logging is OFF
response logging is OFF
memory profiling is INACTIVE
recursive clients: 0/900/1000
recursive high-water: 0
tcp clients: 0/150
TCP high-water: 0
server is up and running
```

# Test your new resolver

Run the following commands and see if you receive answers:

1. dig @localhost    com. SOA +noall +answer
1. dig @100.100.X.67 com. SOA +noall +answer
1. dig @100.100.X.68 com. SOA +noall +answer

The first command should always succeed. If this is your first resolver install 
one of the other commands should fail.

# Restore resolv.conf

Restore `/etc/resolv.conf` to its original content:
```
$ sudo mv /etc/resolv.conf.orig /etc/resolv.conf
```

# Test your resolver again

```
dig com. SOA
```

Look at the output. The important part is `;; SERVER: 100.100.X.67#53(100.100.X.67) (UDP)`

# Done

Please return to [DNS 01 - Resolving](DNS%2001%20-%20Resolving.md) and continue with the lab.
