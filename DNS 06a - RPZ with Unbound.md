# RPZ with Unbound

We have already setup a RPZ. You "just" need to configure it in your resolver.

On the machine with unbound installed the RPZ needs to be configured.
```
sudo nano /etc/unbound/unbound.conf
```
Please add the following section
```
rpz:
    name: "rpz."
    master: 100.64.0.54
    zonefile: "/var/lib/unbound/rpz.zone"
    rpz-log: yes
    rpz-log-name: "RPZ"
```
And in the server section add the following line
```
    module-config: "respip validator iterator"
```
Almost done, we just need to prepare for the zone file transfer
```
sudo mkdir -p /var/lib/unbound
sudo touch /var/lib/unbound/rpz.zone
sudo chown -R unbound:unbound /var/lib/unbound
```
Check if everything is configured correct
```
sudo unbound-checkconf /etc/unbound/unbound.conf
```
And then restart the resolver
```
sudo unbound-control reload
```
Now let's test if it is working
```
sudo unbound-control list_auth_zones
```
Should show the rpz zone like
```
rpz.	serial 1
```

