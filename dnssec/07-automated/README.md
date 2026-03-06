---
layout: page
title: "Automated DNSSEC"
parent: "DNSSEC"
nav_order: 7
---

# Automated DNSSEC

Automated DNSSEC is a collection of different techniques that allow to manage DS records and in many cases even NS records through DNS.

The client has to publish CDS and CDNSKEY records. The parent will automatically pick up the information and make the required changes. The client again needs to monitor when the parent has made changes to move on in the process it is running.

We will make another KSK Rollover, but this time it will be automatic. 
