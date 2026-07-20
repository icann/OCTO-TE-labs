---
layout: page
title: "DNS64"
parent: "DNS"
nav_order: 8
---

# DNS64

# Enable DNS64 on resolv1
# Test
- compare results for AAAA queries for %DOMAIN% and ipv4only.%DOMAIN% between resolv1 and resolv2

# Enable DNS64 on resolv2

# Testing

```
dig ipv4only.%DOMAIN% A
```
```
dig ipv4only.%DOMAIN% AAAA
```
```
dig ipv6only.%DOMAIN% A
```
```
dig ipv6only.%DOMAIN% AAAA
```
```
curl -4 http://%IPv4%
```
```
curl -6 http://[%IPv6%]
```
```
curl -6 http://[64:ff9b::%IPv4%]
```
```
curl http://ipv6only.%DOMAIN%
```
```
curl http://ipv4only.%DOMAIN%
```

