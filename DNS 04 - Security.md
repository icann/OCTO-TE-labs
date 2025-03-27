# Name Server Security

Anything connected to the internet is subjected to a constant barrage of hacking or abuse attempts.
For new wordpress sites it takes on average below 10 seconds for the first attack to arrive.
DNS servers are not long behind. Therefore it is of critical importance to harden your servers before
they go online.

## Hardening the OS

First step in any security for applications is a secure operating system. Security is build in layers and 
if the foundations aren't safe, it isvery hard (or impossible) to mitigate that.

Hardening the OS, starts with making sure that no unneccessary ports are open, ssh only accepts certificates,
only users can login (no remote root access) and many other security best practices. Hopefully you or your
company has already routines in place for hardening your machines.

Hardening the OS is beyond this course, but it is the foundation of all further steps.

## Hardening Authoritative DNS Servers

The DNS has many attack vectors, from spoofing to DDoS and beyond. But why spoof a response when you easily 
can change data on the autoritative server? So hardening and protecting your DNS infrastructure is 
important for the security of your IT infrastructure. DNS is used to secure email, all certificates and 
even many firewalls use DNS based rules.

Please follow theses two labs to secure your lab infrastructure. Afterwards return to this document and go to the next section.

[DNS 04a - Primary Security](DNS%2004a%20-%20Primary%20Security.md)
[DNS 04b - Secondary Security](DNS%2004a%20-%20Secondary%20Security.md)

## Hardening Resolvers

