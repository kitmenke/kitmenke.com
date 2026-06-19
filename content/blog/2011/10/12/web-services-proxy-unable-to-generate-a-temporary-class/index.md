---
author: Kit
categories:
  - Uncategorized
date: 2011-10-12T14:09:15Z
guid: http://kitmenke.com/blog/?p=395
id: 395
tags:
  - dotNET
  - WebServices
title: "Web services proxy: Unable to generate a temporary class"
---

I ran across a strange error while deploying a very simple DLL. It contained only one class; a web services proxy class to the Lists SharePoint web service that I generated using the [wsdl](http://msdn.microsoft.com/en-us/library/d2s8y7bs%28v=vs.80%29.aspx) tool. Here is the error:

> Unable to generate a temporary class (result=1).
  
> error CS2001: Source file &#8216;C:\WINDOWS\TEMP\_zm3ve2y.0.cs' could not be found
  
> error CS2008: No inputs specified

Apparently, this is because the generated proxy class uses XmlSerializer somewhere internally. [My guess is that it is de-serializing the soap messages into objects](http://stackoverflow.com/questions/3417412/why-is-asp-net-trying-to-generate-a-temporary-class-for-a-web-service-reference)?

Some googling turned up the common answer of &#8220;just give the account access to the temp directory.&#8221; This seemed strange to me (_also something our infrastructure team was wary of doing_). Eventually I found [this awesome answer by Richard](http://stackoverflow.com/questions/657993/system-invalidoperationexception-unable-to-generate-a-temporary-class-result-1/658015#658015). Essentially your options are:

  1. Give the account access to the temp directory
  2. Use the sgen tool to pre-generate the serialization assembly
  3. Not use the proxy class (??)

In my opinion, using sgen is the logical choice. The [documentation for sgen](http://msdn.microsoft.com/en-us/library/bk3w6240%28VS.80%29.aspx) also puts it pretty clearly:

> When the XML Serializer Generator is not used, a **XmlSerializer** generates serialization code and a serialization assembly for each type every time an application is run. To improve the performance of XML serialization startup, use the Sgen.exe tool to generate those assemblies the assemblies in advance. These assemblies can then be deployed with the application.

So all I needed to do was run this command against the Release version of my DLL:

```
sgen /a:MyAssembly.dll /compiler:/keyfile:../../MyAssembly.snk
```

This will generate another assembly: MyAssembly.XmlSerializers.dll. Cool!