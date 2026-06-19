---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-07-09T14:58:40Z
excerpt: When creating a workflow using SharePoint Designer, oftentimes it is useful
  to be able to create a link to the current list item. Unfortunately, the built in
  columns are not so helpful.
guid: http://kitmenke.com/blog/?p=74
id: 74
tags:
- SharePoint 2007
- SharePoint Workflow
title: Creating a link to the current item in a workflow
---

When creating a workflow using SharePoint Designer, oftentimes it is useful to be able to create a link to the current list item. Unfortunately, the built in columns are not so helpful.

Below are a couple of the columns that are found in the Current Item:

[<img class="alignnone size-full wp-image-75" title="Workflow String Builder URL columns" src="/uploads/2009/07/workflow_list_url.png" alt="Workflow String Builder URL columns" width="500" height="421" srcset="/uploads/2009/07/workflow_list_url.png 500w, /uploads/2009/07/workflow_list_url-300x252.png 300w" sizes="(max-width: 500px) 100vw, 500px" />](/uploads/2009/07/workflow_list_url.png)

Here is the result after running the workflow for an item:

> Encoded Absolute URL:
  
> http://server/sites/SiteCollection/Workflow/Lists/RandomTester/1_.000
> 
> Server Relative URL:
  
> /sites/SiteCollection/Workflow/Lists/RandomTester/1_.000
> 
> Path:
  
> sites/SiteCollection/Workflow/Lists/RandomTester
> 
> URL Path:
  
> /sites/SiteCollection/Workflow/Lists/RandomTester/1_.000  

 All of these URLs are pretty much useless except for Path since they all have &#8220;1_.000&#8221; appended on the end of them. I think this has something to do with the version number... but why this is on the end makes no sense to me.

What this means then, is that you have to create a URL to the item.

You can copy a link to the DispForm.aspx page and then use the current item's ID:

> http://server/sites/SiteCollection/Workflow/Lists/RandomTester/DispForm.aspx?ID=[]%RandomTester:ID%]