---
layout: page
title: "Secondaries"
parent: "Reverse DNS"
nav_order: 2
---

# Configure the secondary authoritative servers (ns1 and ns2)

These servers expose our reverse zone publicly.

You should now know how to configure secondary authoritative servers. If you forgot, go back to the lab where you created the forward zone for your `grp%GRP%.%DOMAIN%` domain and follow the instructions to configure `ns1` and `ns2` as secondary servers for:

```
%GRP%.100.100.in-addr.arpa.
```

Once you are done, test that the reverse zone has propagated to both authoritative servers.

# Test your zone configuration and propagation

Use `dig` to verify your zone configuration and propagation.

From your client, run the following queries:

```
dig @100.100.%GRP%.66  -x 100.100.%GRP%.66
dig @100.100.%GRP%.130 -x 100.100.%GRP%.66
dig @100.100.%GRP%.131 -x 100.100.%GRP%.66
```

All queries should return the same answer.

If they don't, review your configuration before continuing.

Try the same queries for one or two other groups in the class and compare the results.
