# Resolving

The first contact to DNS for most (all) people is resolving.
Using a domain name as adress in a browser or an email. 

Without access to a functioning resolver clients can not use the DNS and in
todays world that means they are for all practical puposes offline. Running a resolver 
has therefore become important and with follow requirements of 100% uptime.

In this lab we will probably not achieve a 100% uptime, but we will look at
resolver installations, how to secure them and how to make them more resilient.

Go to your cli machine and try:
```
ping icann.org
```
Your poutput probably looks something like
```
ping: icann.org: Temporary failure in name resolution
```
This is of course because we do not have any resolver yet.

On most systems, especially any linux based systems, the contents of the `/etc/resolv.conf`
file decide which resolver gets used. Please have a look at that file. It's contents should 
look something like
```
search grpX.lab_domain
nameserver 100.100.X.67
nameserver 100.100.X.68
nameserver fd89:59e0:X:64::67
nameserver fd89:59e0:X:64::68
```

So our first order of business is to fix resolving.

Please chose two of the following labs for your machines resolv1 and resolv2

[Resolver Setup with Bind](DNS%2001a%20-%20Bind.md)
[Resolver Setup with Unbound](DNS%2001b%20-%20Unbound.md)

# Testing

Now that you have setup two resolvers, resolving should be working on your cli machine.

Please run the following tests:

1. dig com SOA 
1. dig icann.org AAAA
1. dig icann.org MX
1. dig icann.org TXT
1. dig org NS
