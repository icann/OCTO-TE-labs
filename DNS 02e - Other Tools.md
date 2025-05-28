# Other Tools

Here are a few hounorable mentions of tools that can be useful in DNS server
management and trouble shooting.
These tools are very powerful and if used wrong can themselves become a problem.
So be very careful!

# DNSTOP

`dnstop` listens on an interface and collects statistics on DNS traffic. 
Usualy you will have a port mirroring enabled and then collect statistics
on another device (not your DNS server)

# DNSPERF

> dnsperf  is  a  DNS  server  performance  testing tool.  It is primarily intended 
> for measuring the performance of authoritative DNS servers, but it can also be 
> used for measuring caching server performance in a closed laboratory environment.  

```
curl -s -O https://data.iana.org/TLD/tlds-alpha-by-domain.txt
perl -n -e 'chomp; print"$_ DNSKEY\n" if !m/#/;' tlds-alpha-by-domain.txt > data.file
dnsperf -s 100.100.X.68  -d data.file 
dnsperf -s fd89:59e0:X:64::68  -d data.file 
```

# TCPDUMP 

When everything else fails, `tcpdump`will show you what's going on one the wire.

# Wireshark

`wireshark` is tcpdump best friend. Makes the collected data so much more readable.


