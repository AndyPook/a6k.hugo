---
title: WebAPI - AddWithoutValidation method not found
date: 2012-08-24T10:09:00Z
summary: If you are using WebAPI and have recently installed VS2012. Your WebAPI stuff will be broken. Your controller method will be called then you’ll just get a 500 response.
---

If you are using WebAPI and have recently installed VS2012. Your WebAPI stuff will be broken.
Your controller method will be called then you’ll just get a 500 response. Poking VS got it to give up the underlying exception which is "Method not found System.Net.Http.HttpHeaders.AddWitoutValidation".

Several hours of spelunking, trying framework source stepping, break on exception, beating it with a big stick and compiling from a command line with “msbuild /v:d” which shows assembly reolution resulted in realising that VS was compiling against the correct assemblies (ie I’d previously grabbed the RC from nuget). But…

Using the Fusion Log Viewer (fuslogvw) showed that when I ran the project the System.Net.Http assembly was being redirected to the new framework version instead of the file reference to my copy of the dll.

Here’s my solution: Add your own assembly redirect to ensure the right version of the assembly is used.
Simply add the following section to your app.config (if you are self hosting) or to web.config (if you are hosting in IIS).

```xml
<runtime>
  <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
    <dependentAssembly>
      <assemblyIdentity name="System.Net.Http" publicKeyToken="b03f5f7f11d50a3a" culture="neutral" />
      <bindingRedirect oldVersion="1.0.0.0 - 2.0.0.0" newVersion="2.0.0.0"/>
    </dependentAssembly>
  </assemblyBinding>
</runtime>
```

Some will ask “Why bother? Why not just use the RTM?”. Well, I have a new version of our product about to RTM. I really don’t want to have to refactor a bunch of code and have it all go back through QA and regression testing.
We will move on to WebAPI RTM but not until our next version.

This is the risk that the decision to make 4.5 an in-place upgrade exposes us to. What other subtle changes in the framework are there?

I’m glad we test and deploy from build servers and not from a developers machine. You should check that your build environment is compatible with your production/runtime environment.
