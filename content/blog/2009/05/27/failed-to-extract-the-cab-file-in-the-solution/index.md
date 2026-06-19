---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-05-27T13:49:31Z
guid: http://kitmenke.com/blog/?p=33
id: 33
tags:
- Deployment
- MOSS2007
- SharePoint 2007
- Solution
title: Failed to extract the cab file in the solution.
---

Got the following message when trying to deploy a solution file to the server:

> Failed to extract the cab file in the solution.

Luckily [Robert Bogue](http://thorprojects.com/blog/archive/2008/01/21/stsadm-strikes-again-failed-to-extract-the-cab-file-in-the-solution-.aspx) had already found the answer: we had duplicated an entry for a web part in our solution's .ddf file. Apparently, makecab allows you to have the same file in the same location multiple times. I'm not really sure why it doesn't just overwrite it, but in any case, the solution doesn't know what to do with it either.