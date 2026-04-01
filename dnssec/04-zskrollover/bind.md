---
layout: page
title: "Bind"
parent: "ZSK rollover"
nav_order: 1
---

# ZSK rollover

> [!TIP]
> This lab will only work when you have done the previous lab "Manual Signing".

Before you continue, check that:

```
$ dig grp%GRP%.%DOMAIN% dnskey +dnssec +multiline
```
shows you one ZSK and one KSK. Remember that the KSK has flags 257.

```
$ dig grp%GRP%.%DOMAIN% SOA +dnssec +multiline
```
gives you the SOA record for your domain, and that it is signed.

> [!WARNING]
>
> If any of these things are not true, finish off the previous labs before you start this one. If you need help getting up-to-date, one of your friendly workshop lab staff will be happy to assist.

## Introduction

In this lab, we will carry out a manual key rollover of the ZSK. Since we are not changing the KSK, we do not need to generate a new DS record to the parent zone (this will be done in the case of a KSK rollover). 

In this lab we are using the ZSK rollover methodology which is called double-signature. Pre-publication is another possible methodology, but we will not use it.

Out steps will be:

* Create a new ZSK in addition to the existing one (a “**successor** ZSK”).
* Publish both ZSK and sign with both ZSK
* Wait for TTL timeout 
* Remove the old ZSK and sign only with the new ZSK

## ZSK Rollover

Find your current ZSK in the keys folder. The ZSK should have the value 256. That's the one we want. Note down its number and have a look in the content of the file. In our example, this is the ZSK. 

> [!CAUTION]
>
> Remember that your filename will be different! Do not simply cut and paste Kmytld.+008+26734 later in the lab!

```
cat Kgrp%GRP%.%DOMAIN%.+???+?????.key 
```

```
; This is a zone-signing key, keyid ?????, for grp%GRP%.%DOMAIN%.
; Created: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Publish: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Activate: 20221103220010 (Thu Nov  3 22:00:10 2022)
grp%GRP%.%DOMAIN%. IN DNSKEY 256 3 8 AwEAAadehqG2E23DsA4MnHcaeTH/bKTHlLftvUKR9i8lVbvWNTydacdQ MsZJPTTFZXHeXFdSmxAxImc/FEGNnk9VRr3FfzfJKbc+s6r17PLWn1bO sUxawKZogOvISPytMcWnhbj8Trs8KOoAekB1PRaiPGsCP/nj68ufvrzl x2AcfDJAWPynNDjgHxeFygifVlM6iYuzmPlpcMAY5LCIS/B1MrfashJh wtj0dldgqJSp6yZHaP8vcrMa6+s5McQcqRpyoR2rpNpl6PiOUBtjE0Ho nwg1XYzSaBAbhLdmQhC4MWL/aNiXp1ybwXSVb8uZqL5k26QlKRNH2eB8 
YRRtq+B9rIs=
```

### Create a new ZSK (a “successor” ZSK)

We do not need to specify the full set of parameters (algorithm name, key size, etc.) when we generate a replacement ZSK  because we will tell the dnssec-signzone command that we are creating a successor to the old ZSK, and the software will make sure the new key it generates matches.

```
sudo dnssec-keygen -f ZSK -a ECDSAP256SHA256 -K /var/lib/bind/keys grp%GRP%.%DOMAIN%
```

```
Generating key pair............+++++ ..............................+++++ 
Kgrp%GRP%.%DOMAIN%.te-labs.training.+013+12969
```
Change the ownership of the new file and reload bind.
```
sudo chown -R bind:bind /var/lib/bind/keys
```

### * Publish both ZSK and sign with both ZSK

1. Edit the zone and increase the serial
2. Resign the zone
```
sudo dnssec-signzone -S -K /var/lib/bind/keys -o grp%GRP%.%DOMAIN% /var/lib/bind/zones/db.grp%GRP%
```  
Output should be something like 
```
Fetching grp%GRP%.%DOMAIN%/ECDSAP256SHA256/14800 (ZSK) from key repository.
Fetching grp%GRP%.%DOMAIN%/ECDSAP256SHA256/65181 (ZSK) from key repository.
Fetching  grp%GRP%.%DOMAIN%/ECDSAP256SHA256/16579 (KSK) from key repository.
Verifying the zone using the following algorithms:
- ECDSAP256SHA256
Zone fully signed:
Algorithm: ECDSAP256SHA256: KSKs: 1 active, 0 stand-by, 0 revoked
                            ZSKs: 2 active, 0 stand-by, 0 revoked
/var/lib/bind/zones/db.grp%GRP%.signed
```
### Wait for TTL timeout

In this lab timeouts are very short, you can proceed immediately. But on the internet this can easily be one or two days you'll have to wait.

### Remove the old ZSK and sign only with the new ZSK

Hopefully you remember which of the files is the new and which is the old ZSK.

```
mv /var/lib/bind/keys/Kgrp%GRP%.%DOMAIN%.te-labs.training.+013+?????.key /var/lib/bind/keys/old_Kgrp%GRP%.%DOMAIN%.te-labs.training.+013+?????.key
mv /var/lib/bind/keys/Kgrp%GRP%.%DOMAIN%.te-labs.training.+013+?????.private /var/lib/bind/keys/old_Kgrp%GRP%.%DOMAIN%.te-labs.training.+013+?????.private
```
and now we sign again, but we will increase the serial before we do that
```
sudo dnssec-signzone -S -K /var/lib/bind/keys -o grp%GRP%.%DOMAIN% /var/lib/bind/zones/db.grp%GRP%
```  

> [!IMPORTANT]
>
> In real life, you should anticipate all these changes and set enough time between events. This will allow you to react properly in case of any unexpected behavior.
>
> It is also critical to have a proper monitoring system and maintenance routines in place to watch these events in details and take necessary actions when and where required.
