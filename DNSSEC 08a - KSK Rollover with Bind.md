# KSK Rollover

Currently DNSSEC for your zone is automatically handled by bind. We will now bind tell to roll keys.

A KSK rollover is a somewhat more complicated procedure. We will use the following method:

1. Create a new key and publish it in the zone
1. Use the new key and the old key to sign the DNSKEY set
1. Wait for TTL timeout
1. Switch from old to new DS record

## Creating a new key

As bind handles all DNSSEC we will not create the key ourself, but just tell bind about our intent to roll the key. For this we need the keytag of our KSK.

```
dig grpX.lab_domain DNSKEY +noall +answer +multi
```
Should look something like
```
grpX.lab_domain. 5 IN DNSKEY 257 3 13 (
        PnpghNjNXfy29eidzxizUcAwtdrQ7ylXbcdsmyhSXrW1
        4n5PZYdnZAEuIubnm6GZoF+p5BX1CuAXwUg+Ela2Eg==
    ) ; KSK; alg = ECDSAP256SHA256 ; key id = 3963
grpX.lab_domain. 5 IN DNSKEY 256 3 13 (
        EbjyFdlDwCAYIqZlBiBmvzfQzqag1TM1wIkGDXAdEPoe
        vPmIPe2+4FOD9AEQCbVUxRvpsosN4rThXTgJk1XPaA==
    ) ; ZSK; alg = ECDSAP256SHA256 ; key id = 43395
```
We need the key id of the key with the flag value of 257. (In this example 3963)

Now on the soa machine we run
```
sudo rndc dnssec -rollover -key ???? grpX.lab_domain
```
Please run
```
sudo rndc dnssec -status grpX.lab_domain
```
A new key of type KSK should be in the list and it should already be published.
```
dig grpX.lab_domain DNSKEY +dnssec +noall +answer
```
Should result in something like this
```
grpX.lab_domain. 5   IN      DNSKEY  257 3 13 PnpghNjNXfy29eidzxizUcAwtdrQ7ylXbcdsmyhSXrW14n5PZYdnZAEu Iubnm6GZoF+p5BX1CuAXwUg+Ela2Eg==
grpX.lab_domain. 5   IN      DNSKEY  256 3 13 EbjyFdlDwCAYIqZlBiBmvzfQzqag1TM1wIkGDXAdEPoevPmIPe2+4FOD 9AEQCbVUxRvpsosN4rThXTgJk1XPaA==
grpX.lab_domain. 5   IN      DNSKEY  257 3 13 tZnNamGitG3rL3H3JiA0YCPalEtLYapYvON5xC3ozkfjcSG5c4CsvjB4 /iWeSepoHDXeJxoVWP+F2UULDNEm3A==
grpX.lab_domain. 5   IN      RRSIG   DNSKEY 13 4 5 20260203193739 20260203182740 3963 grpX.lab_domain. ASmUp90Pyi53omghejWUcud06P+2CAC4ln7SPgYxOzvKVBzDQ70G1oHl qiWPI1sUn1IbofzerOmXRKCSwPRW0g==
grpX.lab_domain. 5   IN      RRSIG   DNSKEY 13 4 5 20260203193739 20260203182740 32320 grpX.lab_domain. 9ykAl8NBXMivhSb4A1ol5Dvjsaq9uZYDUnPEWUYBslHPaYMsgOXzU9Ns mEng3tcvaBl7ZYsDeLD0/GyvWhCdHw==
```
There should be three keys and two RRSIG records.

We are using extremly short TTLs, so we can update the DS record right away.

```
dig @localhost grpX.lab_domain DNSKEY | dnssec-dsfromkey -f - grpX.lab_domain
```
And use the labs web page to update the record.

Now check if you see the new DS record in the parent.
```
dig grpX.lab_domain DS
```

Once the DS record is update we can tell bind that the DS record is updated.
```
sudo rndc dnssec -checkds -key old_id withdrawn grpX.lab_domain
sudo rndc dnssec -checkds -key new_id published grpX.lab_domain
```
If you repeatedly run 
```
sudo rndc dnssec -status grpX.lab_domain
```
You can see the status of the keys changing. Once hte new key is in status "omnipresent"  run
```
dig @100.100.X.66 grpX.lab_domain DNSKEY +dnssec +noall +answer
```
Now you should get only two keys and one RRSIG record.

