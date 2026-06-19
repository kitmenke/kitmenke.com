---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-05-21T10:48:28Z
guid: http://kitmenke.com/blog/?p=18
id: 18
tags:
- SharePoint Search
- SharePoint 2007
title: Enterprise Search and SharePoint
---

I've learned a couple of different things from messing with Enterprise Search and SharePoint. A lot of this has to do with the initial setup and how different crawls affect the index.

What this assumes:

  * You know how to get to Central Admin and the Search Admin screens
  * Any URLs below you will have to replace &#8220;centraladmin&#8221; with your own server ip
  * The place where you can run incremental or full crawls is:
  
    http://centraladmin/ssp/admin/_layouts/listcontentsources.aspx

Metadata properties:

  * In order to have more columns to search, you must add them to the &#8220;Metadata&#8221; in the ssp
  
    http://centraladmin/ssp/admin/_layouts/schema.aspx
  * Any metadata property has a 64 character limit when querying using Enterprise Search SQL (ESSQL)
  * In order for a column to show up in the Add Mapping dialog an **<span style="text-decoration: underline;">INCREMENTAL</span>** crawl is required (column existed and just didn't have data in... added some data to the list making sure to populate the new fields and then ran an incremental)
  * Adding a Managed Property requires a **<span style="text-decoration: underline;">FULL</span>** crawl in order to populate the data in that field

Search visibility:

  * If a site has Search Visibility disabled, it will show up as a warning in the crawl log. LISTS DO NOT
  * Changing a list's Search Visibility (in advanced settings) will require a **<span style="text-decoration: underline;">INCREMENTAL</span>** (full is not required) crawl in order to start showing up in search results

Search scopes:

  * Changing an existing search scope requires you to update the scope again
  
    Go back to Central Admin -> ssp -> Search Settings -> Start Updating
  
    After clicking Update... it almost always goes from 0% to 100% after a while... no in between
  * Changing scopes does not require an incremental or full crawl (you simply need to update the scope again) see above
  * For a search scope, to have it include list items your scope should have a &#8220;folder&#8221; with a value like (encoded url with trailing slash):
  
    http://moss/sitecollection/subsite/Lists/My%20List%20Name/

Also, here is some very **unhelpful** microsoft documentation:
  
<http://msdn.microsoft.com/en-us/library/ms493660.aspx>

Leave a comment if you have questions!