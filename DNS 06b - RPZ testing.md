# RPZ Testing

> [!IMPORTANT]
> On your resolv1 one you should have a resolver with configured RPZ running. 
> And on your resolv2 should be a resolver without configured RPZ running. 

## RPZ Policies

An RPZ can specify different responses for queries
1. answer with NXDOMAIN
1. answer with NODATA (NOERROR, empty answer section)
1. drop the query, no answer is returned
1. force the client to use TCP
1. rewrite the answer to a different value

Matching to a query can be done on several attributes
1. query name
1. response value 
1. client ip
1. name server name
1. name server ip

Below we will explore some of those configurations.

## RPZ testing

For all tests below, please compare the answers between resolv1 and resolv2.

### NXDOMAIN
In the RPZ zone file we find the following line
```
nxdomain.internal          CNAME   .
```
Let's check if what answers look like
```
dig @100.100.X.68 nxdomain.internal TXT
dig @100.100.X.67 nxdomain.internal TXT
```
The one dig command should give you data like
```
;; ANSWER SECTION:
nxdomain.internal.	30	IN	TXT	"Block me if you can"
```
The second should return a NXDOMAIN answer.

### NODATA
In the RPZ zone file we find the following line
```
nodata.internal            CNAME   rpz-nodata.
```
Let's check if what answers look like
```
dig @100.100.X.68 nodata.internal TXT
dig @100.100.X.67 nodata.internal TXT
```
The one dig command should give you data like
```
;; ANSWER SECTION:
nodata.internal.	30	IN	TXT	"Block me if you can"
```
The second should return a NOERROR answer with an empty answer section.

### DROP
In the RPZ zone file we find the following line
```
drop.internal CNAME   rpz-drop.
```
Let's check if what answers look like
```
dig @100.100.X.68 drop.internal TXT
dig @100.100.X.67 drop.internal TXT
```
The one dig command should give you data like
```
;; ANSWER SECTION:
drop.internal.		30	IN	TXT	"Block me if you can"
```
The second should something like
```
;; communications error to 100.100.X.67#53: timed out
```

### TCP-ONLY
In the RPZ zone file we find the following line
```
tcponly.internal CNAME   rpz-tcp-only.
```
Let's check if what answers look like
```
dig @100.100.X.68 tcponly.internal TXT
dig @100.100.X.67 tcponly.internal TXT
```
This you should look at the comments at the end of the `dig` output. 
It indicates if the result was obtained through UDP or TCP.
```
;; SERVER: 100.100.X.67#53(100.100.X.67) (TCP)
```

### WILDCARD
In the RPZ zone file we find the following line
```
*.wildcard.internal CNAME   .
```
Let's check if what answers look like
```
dig @100.100.X.68 wildcard.internal TXT
dig @100.100.X.67 wildcard.internal TXT
dig @100.100.X.68 any-name-you-want.wildcard.internal TXT
dig @100.100.X.67 any-name-you-want.wildcard.internal TXT
```
Wildcard blocking will not block the exact label. That will have to be blocked with it's own blocking instructions.

### PASSTHRU
In the RPZ zone file we find the following line
```
override.wildcard.internal         CNAME   rpz-passthru.
```
This “whitelist” example disables policy for this name.
```
dig @100.100.X.68 override.wildcard.internal TXT
dig @100.100.X.67 override.wildcard.internal TXT
```
### LOCAL-DATA
In the RPZ zone file we find the following line
```
rewrite.internal            TXT "A MESSAGE FROM RPZ"
```
Returns different data instead of the original answer.
```
dig @100.100.X.68 rewrite.internal TXT
dig @100.100.X.67 rewrite.internal TXT
```

### IP-ADDRESSES
In the RPZ zone file we find the following lines
```
32.2.100.100.100.rpz-ip        CNAME   .
128.dad.bad.zz.db8.2001.rpz-ip CNAME   .
```
Answers which result in these IP addresses will be answered with NXDOMAIN.

Note the address notation. First is the prefix length followed by the ip address in reverse notation.

```
dig @100.100.X.68 v4blocked.internal A
dig @100.100.X.67 v4blocked.internal A
dig @100.100.X.68 v4blocked.internal AAAA
dig @100.100.X.67 v4blocked.internal AAAA
dig @100.100.X.68 v6blocked.internal A
dig @100.100.X.67 v6blocked.internal A
dig @100.100.X.68 v6blocked.internal AAAA
dig @100.100.X.67 v6blocked.internal AAAA
```

### NSDNAME
In the RPZ zone file we find the following line
```
badns.internal.rpz-nsdname  CNAME   rpz-drop.
```
The query will be droped if a name of a nameserver in the delegation matches.
```
dig @100.100.X.68 badnsname.internal TXT
dig @100.100.X.67 badnsname.internal TXT
```
Use `dig` to investigate the configuration of `badnsname.internal`. Which nameserver 
is configured?

### NSIP
In the RPZ zone file we find the following line
```
32.3.100.100.100.rpz-nsip   CNAME   rpz-drop.
```
The query will be droped if the ip of a nameserver in the delegation matches.
```
dig @100.100.X.68 evilnsip.internal TXT
dig @100.100.X.67 evilnsip.internal TXT
```
Use `dig` to investigate the configuration of `evilnsip.internal`. Which nameserver 
is configured? What is it's ip address?

### Logging

Have a look at `/var/log/rpz.log` or `/var/log/unbound/rpz.log`. You should find 
all the above queries in the log.
