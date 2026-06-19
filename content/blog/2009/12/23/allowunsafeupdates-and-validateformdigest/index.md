---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-12-23T11:34:41Z
guid: http://kitmenke.com/blog/?p=175
id: 175
tags:
- SharePoint 2007
title: AllowUnsafeUpdates and ValidateFormDigest
---

Add this to the list of things every SharePoint developer should know (up there with disposing SPWebs and SPSites).

In general...

  1. Don't update SharePoint objects on a GET request
  2. Call SPUtility.ValidateFormDigest() before anything on a POST request

Here are the two links to read:

  * <a rel="bookmark" href="http://hristopavlov.wordpress.com/2008/05/16/what-you-need-to-know-about-allowunsafeupdates/">What you need to know about AllowUnsafeUpdates (Part 1)</a>
  * <a rel="bookmark" href="http://hristopavlov.wordpress.com/2008/05/21/what-you-need-to-know-about-allowunsafeupdates-part-2/">What you need to know about AllowUnsafeUpdates (Part 2)</a>