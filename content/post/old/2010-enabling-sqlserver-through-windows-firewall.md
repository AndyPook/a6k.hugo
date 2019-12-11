---
title: Enabling SQL Server access through the Windows firewall
date: 2010-03-08T12:47:00Z
summary: Opening ports via the command line
---

This is one of the annoying things about setting up a new machine. You think that you’ve got all the system stuff installed. Application deployed. Everything seems good. Then, later, you try to resolve some issue by connecting from some other machine (like you’re dev machine) and it won’t let you. The firewall is blocking remote connections. You know you should “do the right thing” and put in a very specific rule to just allow SQL Server traffic but you need to get this done _now_ so you just disable the firewall to get your problem fixed.

Do you remember to re-enable it???

So here’s how you do it the command line way.

```
netsh advfirewall firewall add rule name = SQLPort dir = in protocol = tcp action = allow localport = 1433 remoteip = localsubnet profile = DOMAIN
```

Copy it to a bat file somewhere then run it on whatever machines you need access to.

Taken from http://msdn.microsoft.com/en-us/library/cc646023.aspx

 