# Reverse DNS

Everybody knows that the DNS can be used to translate names to ip numbers. But actually, the DNS can also translate IP numbers to names. This is done with the Reverse DNS.

# Configure the primary authoritative server (SOA)

To map your IP address to your domain name, we’ll need to setup a reverse zone. 
We are going to configure a hidden authoritative server for your reverse zone 
and create the authoritative zone reverse\_grp***X***.***lab_domain***.

```
# nano /etc/bind/zones/reverse_grpX.lab_domain.te-labs.training
```


```
$TTL    300
@		IN		SOA		soa.grpX.lab_domain. dnsadmin.lab_domain. (                                            
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;

                IN              NS              ns1.grpX.lab_domain. ; your name server
                IN              NS              ns2.grpX.lab_domain. ; your name server
66		IN		PTR		soa.grpX.lab_domain.
67		IN		PTR		resolv1.grpX.lab_domain.
68		IN		PTR		resolv2.grpX.lab_domain.
130		IN		PTR		ns1.grpX.lab_domain.
131		IN		PTR		ns2.grpX.lab_domain.
```

Save and exit.

Run the following command to check for any errors in your setup:

```
# named-checkzone X.100.100.in-addr.arpa /etc/bind/zones/reverse_grpX.lab_domain.
```

Next, edit the /etc/bind/named.conf.local file and add the following lines:

```
zone "X.100.100.in-addr.arpa" {
  type primary;
  file "/etc/bind/zones/reverse_grpX.lab_domain.";
  allow-transfer { any; };
  also-notify {100.100.X.130; 100.100.X.131; };
};
```

Save and exit.

Run the following command to check for any errors in your setup:

```
# named-checkconf
```

Restart Bind 9 and test your reverse DNS using dig

```
# dig -x 100.100.X.66 @localhost
```

Or

```
# dig 66.X.100.100.in-addr.arpa. PTR @localhost
```


Question: Do you get a DNS response with the PTR record in the answer section?



## Configure the secondary authoritative servers (ns1 and ns2) 

These servers are the ones that expose our (reverse) zone publicly (so they will be open-to-all servers). You should now know how to configure the secondary NS. If you forgot, go back to the lab where you created the forward zone for your grpX and follow the instructions to do this new configuration for your reverse zone.

Once you are done with configuration, test your reverse zone propagation.

## Test your zone configuration and propagation.
Use *dig* tool to verify your zone configuration and propagation, then do the same for one or two other groups in the class and share comments. From your client, run the following dig queries. All should return answer otherwise you should review your configurations before continiuing:

1. dig -x 100.100.X.66 @100.100.X.66
2. dig -x 100.100.X.66 @100.100.X.130
3. dig -x 100.100.X.66 @100.100.X.131
4. dig -x 100.100.X.67 @100.100.X.130
5. dig -x 100.100.X.68 @100.100.X.130
