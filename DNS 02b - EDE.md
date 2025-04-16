# Extended DNS Errors (EDE)

A few years back the Internet Engineering Task Force (IETF) standardized
a new way for DNS system to be more specific about errors.

Up to that point many errors were mainly communicated through the status code "SERVFAIL".
But it could mean anything, from non-reachable name servers to DNSSEC validation errors.
So [RFC 8914 Extended DNS Errors](https://www.rfc-editor.org/rfc/rfc8914) fixes this.

The list of all possible EDE values can be found at the [IANA Registry for Extended DNS Error Codes](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#extended-dns-error-codes)

> [!IMPORTANT]
> Not all implementations use these error codes in the same way.

Test the following commands and see which error messages you get.
```
$ dig @100.100.X.67 dnssec-failed.org
$ dig @fd89:59e0:X:64::68 dnssec-failed.org
$ dig @9.9.9.9 dnssec-failed.org
```
