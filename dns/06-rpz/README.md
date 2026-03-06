---
layout: page
title: "RPZ"
parent: "DNS"
nav_order: 6
---

# RPZ

Response Policy Zones (RPZ) are not a very new addition to DNS
but they have come to wider usage only in the last few years.

Response Policy Zones (RPZ) are a way to tell a resolver to block queries or rewrite answers.
There are many different ways to do this. In this lab we will go through a number of them.
But this is by no means a full guide to RPZ usage.

In many countries national certs provide a list of malicious domain names. Often
internet service providers and other network operators receive these lists and
block the contained names in their resolvers. Unfortunately this is often a manual
process, which means it is often very slow.

Please do only one of the following labs. We need one resolver do the blocking and the other to not block.

- [DNS 06a - RPZ with Bind](DNS%2006a%20-%20RPZ%20with%20Bind.md)
- [DNS 06a - RPZ with Unbound](DNS%2006a%20-%20RPZ%20with%20Unbound.md)

Now let's see what RPZ can do

- [DNS 06b - RPZ Testing](DNS%2006b%20-%20RPZ%20Testing.md)

And last but not least let's configure a RPZ. Please use the lab for the same software as you used or DNS 06a.
- [DNS 06c - Local RPZ with Bind](DNS%2006c%20-%20Local%20RPZ%20with%20Bind.md)
- [DNS 06c - Local RPZ with Unbound](DNS%2006c%20-%20Local%20RPZ%20with%20Unbound.md)
