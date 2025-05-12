
# Establish a chain of trust

As a reminder, your domain is grpX.lab_domain, and your parent is lab_domain.

Let's start with looking at the current state of your domain.
Go to [dnsviz.net](https://dnsviz.net) and analyze your domain. 

> [!TIP] If you have analyzed your domain with dnsviz before, make sure to run an new analysis.

Try to follow the chain of trust in the resulting graph.
Take notice of the missing DS in the last step. How can you know that a DS record is not there? How does DNSSEC proof if a DS record exists or not?

Now let's fix this!

## Generate your DS record

Execute the following command to get the DS record

```
# dig @localhost dnskey grpX.lab_domain | dnssec-dsfromkey -f - grpX.lab_domain
```

Your output should look something similar to the following line:

```
grpX.lab_domain. IN DS 12345 8 2 018A86C0139BA5500AC87A5BAD8FB5D8D4F9672C319B34DB5A7F3BC10A424D6E
```

## Push the DS to your parent

Got to your group page on the labs web and submit your DS record in the form.

It will take approx. 2 or 3 minutes to publish the DS record.

## Verify that your DS is published into your parent's zone

Query your parent zone and confirm that they have published your DS.

```
sysadm@cli:~$ dig +nocomments +noall +answer grpX.lab_domain DS
```

Retry until the answer looks like

```
grpX.lab_domain. 60    IN      DS      2404 8 2 8A4D8024E59D115331C8ECAF715E1168A429282646E6861420BEF8D1 7F9676E7
```

Now see if your resolver return the AD flag.

```
dig grpX.lab_domain SOA
```

And finally go back to [dnsviz.net](https://dnsviz.net) and retest your domain. This time a DS record should be shown.

