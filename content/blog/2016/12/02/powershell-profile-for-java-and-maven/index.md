---
id: 840
tags: ["Java", "Powershell"]
title: Powershell profile for Java and Maven
description: An example powershell profile script which sets up the JAVA_HOME and M2_HOME environment variables for Java and Maven development.
author: Kit
categories: ["Uncategorized"]
date: 2016-12-05T12:26:29-06:00
---

As we gradually replace regular windows command line with powershell, it will be useful to set up a powershell environment for Java / Maven development. 

Previously, I had a command file which looked like this:

```cmd
@echo off
set MAVEN_OPTS=-Xmx512m
set MAVEN_VERSION=3.3.9
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_92
set M2_HOME=C:\apache-maven-%MAVEN_VERSION%
set PATH=%M2_HOME%\bin;%JAVA_HOME%\bin;%PATH%
 
echo Java home is %JAVA_HOME%
echo Maven home is %M2_HOME%
 
cd /d C:\git
```

In powershell, all we need to do is [create a powershell profile script](http://www.howtogeek.com/50236/customizing-your-powershell-profile/) and convert our CMD file to powershell:

```powershell
echo "Profile loaded from $profile"
cd C:\git
 
# Set environment variables
$env:MAVEN_OPTS = "-Xmx512m"
$env:MAVEN_VERSION = "3.3.9"
$env:JAVA_HOME = "C:\Program Files\Java\jdk1.8.0_92"
$env:M2_HOME = "C:\apache-maven-$($env:MAVEN_VERSION)"
$env:PATH = "$($env:M2_HOME)\bin;$($env:JAVA_HOME)\bin;$($env:PATH)"
 
echo "Java home is $($env:JAVA_HOME)"
echo "Maven home is $($env:M2_HOME)"
```

Test it out by opening your shell and running `java -version` and `mvn --version`.