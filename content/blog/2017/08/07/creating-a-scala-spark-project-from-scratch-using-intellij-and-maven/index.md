---
tags: ["Hadoop"]
title: Creating a new Spark Project for Scala development using IntelliJ and Maven
author: Kit
categories: ["Hadoop"]
date: 2017-08-06T20:38:16-05:00
draft: true
---

Creating a project for working with Spark and Scala. There are multiple ways to work with Spark but this post outlines my workflow.

Ultimate goal is to create a reproducible dev environment where anyone can clone the repository in git and be up and running quickly.

How to create a project for developing Spark applications on windows in Scala using IntelliJ.  

## Overview

By the end of this guide, you'll have a Spark app developed in Scala and running locally on a Windows machine.

## Prequisites

Install Java 8

## Why?

My opinion for a good local dev environment:

 - IDE: IntelliJ
    - Why? Great IDE, the community edition is free and works really well, 
    - Alternatives: Eclipse, Sublime  TODO: Why?
 - Project management: Maven, Alternatives: SBT, gradle
    - Most big data projects are written in Java and leverage maven
    - SBT is used by the scala community so these project mgmt tools are at odds
 - Scala

## Development Environment Setup

Before we begin, we need to know the correct Spark, Java, and Scala version to use.

If you already have a cluster then this is probably decided for you. Either as your admin or login to the edge node to figure it out.

For my local development on my Windows laptop, I've installed the latest Java 8 JDK and IntelliJ Community Edition 2017.2.

```
> java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

First we'll setup for  Spark 1.6.3 which according to the [Documentation](https://spark.apache.org/docs/1.6.3/) using Scala 2.10.x.

> For the Scala API, Spark 1.6.3 uses Scala 2.10. You will need to use a compatible Scala version (2.10.x).

1. From the Intellij splash screen, click Create New Project
1. Click Maven on the left hand side
1. Under the Project SDK dropdown make sure your JDK is installed. If it isn't there, click new to add it. Example: C:\Program Files\Java\jdk1.8.0_144
1. Click Next - we're not going to select a maven archetype

<a href="#" data-featherlight="/uploads/2017/08/create-maven-project.png"><img src="/uploads/2017/08/create-maven-project.png" /></a>

1. Fill out group id (ex: com.kitmenke.spark), artifact id (ex: spark-example), and version (ex: 1.0-SNAPSHOT).

http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22net.alchim31.maven%22%20AND%20a%3A%22scala-archetype-simple%22

GroupId net.alchim31.maven
ArtifactId scala-archetype-simple
Version 1.6

Maven comes bundled with Intellij so the easiset option is just to use that. Click Next.

http://docs.scala-lang.org/tutorials/scala-with-maven.html

https://github.com/davidB/scala-maven-plugin