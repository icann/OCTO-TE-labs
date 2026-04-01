---
layout: page
title: "Secondary Bind"
parent: "Security"
nav_order: 3
---

# Secondary Security BIND

Currently our secondary servers do not receive any zone updates.
This is due to the fact that we configured our primary to only allow
transfers with TSIG, but havent't configured TSIG on our secondaries.

## Add the TSIG key to your secondary BIND

In **/etc/bind/named.conf.options**, add the tsig key, and a statement to tell which key to use when talking to “100.100.%GRP%.66;” (the soa server ):

```
key "grp%GRP%-key" {
        algorithm hmac-sha256;
        secret "THIS_IS_MY_KEY";
};

server 100.100.%GRP%.66 {		
        keys { grp%GRP%-key; };
};
server fd89:59e0:%GRP%:64::66 {	
        keys { grp%GRP%-key; };
};
```

Save, exit and restart bind9.

## Testing the configuration

On SOA server increase the serial and reload the zone. Then, 
```
sudo rndc reload grp%GRP%.%DOMAIN%
```

In ns1, go to logs and validate that the transfer was successful.

```
tail /var/log/syslog
```
Should look like
```
zone grp%GRP%.%DOMAIN%/IN: Transfer started.
transfer of 'grp%GRP%.%DOMAIN%/IN' from 100.100.%GRP%.66#53: connected using 100.100.%GRP%.13>
zone grp%GRP%.%DOMAIN%/IN: transferred serial 2022052401: TSIG 'grp%GRP%-key'
transfer of 'grp%GRP%.%DOMAIN%/IN' from 100.100.%GRP%.66#53: Transfer status: success
transfer of 'grp%GRP%.%DOMAIN%/IN' from 100.100.%GRP%.66#53: Transfer completed: 1 messag>
```

## Access over DNS

Secondary servers are usually used to answer queries on the internet.
So we can not restrict access to the server by ip address. But we do not 
need to allow zone transfers out.

Please make sure your `named.conf.options` contains in the options section
```
    allow-query { any; };
    allow-transfer { };
    allow-notify { };
    also-notify { };
```
In `named.conf.local` we need to allow notify for our zone grp%GRP%.%DOMAIN% by including the following config in the zone section
```
    allow-notify { 
        100.100.%GRP%.66;
        fd89:59e0:%GRP%:64::66;
    }
```
Currently our server accepts notify message from any source. Attackers could
use this for a resource exhaustion attack. Let's only accept notifies from 
our master servers with the correct keys.
```
    masters { 
        100.100.%GRP%.66 key grp%GRP%-key; 
        fd89:59e0:%GRP%:64::66 key grp%GRP%-key;
    };
```
Please check your configuration and reload the server
```
named-checkconf
sudo rndc reload
```
Now on your soa server increase the serial number for the zone.
Check if the secondary gets and accepts the notify and successfully transfers the zone.
