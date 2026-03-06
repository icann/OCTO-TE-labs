---
layout: page
title: "uDNSSEC 09   Automated DNSSEC"
---

# Automated DNSSEC

Automated DNSSEC is a collection of different techniques that allow to manage DS records and in many cases even NS records through DNS.

The client has to publish CDS and CDNSKEY records. The parent will automatically pick up the information and make the required changes. The client again needs to monitor when the parent has made changes to move on in the process it is running.

We will make another KSK Rollover, but this time it will be automatic. 

Please follow one of labs below depending on which software you chose in lab DNS 03a.
- [DNSSEC 09a - Automatic Signing with Bind](DNSSEC%2009a%20-%20Automated%20DNSSEC%20with%20Bind.md) 