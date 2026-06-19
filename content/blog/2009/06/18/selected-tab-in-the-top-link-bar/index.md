---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-06-18T19:30:17Z
excerpt: The problem started after creating multiple aspx pages within my site and
  then setting up a tab for each page by editing the Top Link Bar (also known as the
  top navigation) under Site Settings. Each of the tabs worked correctly, but after
  clicking a tab that tab would not be highlighted and the home tab would stay selected.
guid: http://kitmenke.com/blog/?p=70
id: 70
tags:
- SharePoint 2007
title: Selected tab in the Top Link Bar
---

I recently ran into the weird issue of not having my tabs highlighted correctly. After consulting Uncle Google, it seems [Andy Burns](http://www.novolocus.com/2009/04/07/highlighting-tabs-in-the-top-navigation/) ran into this same issue.

The problem started after creating multiple aspx pages within my site and then setting up a tab for each page by editing the Top Link Bar (also known as the top navigation) under Site Settings. Each of the tabs worked correctly, but after clicking a tab that tab would not be highlighted and the home tab would stay selected.

The fix, as Andy mentions, is to change your URLs to be relative. After the change, your tabs work as you would expect.