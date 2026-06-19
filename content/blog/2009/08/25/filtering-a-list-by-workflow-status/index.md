---
aktt_notify_twitter:
- true
aktt_tweeted:
- 1
author: Kit
categories:
- SharePoint
date: 2009-08-25T13:25:59Z
guid: http://kitmenke.com/blog/?p=131
id: 131
tags:
- SharePoint 2007
- SharePoint Workflow
title: Filtering a SharePoint list by Workflow Status
---

The most recent problem I ran into was when I tried to filter out items that had a  certain Workflow status in the list. More specifically, I was trying to filter on items that had errored out and had a status of &#8220;Error Occurred&#8221; in the list.
  

My first thought was to try to filter based on the text. Using a contains didn't work; I got a message about you can only use a contains on Single line of text, Multiple line of text, or Choice fields.

The solution is to filter based on the integer value:
  
[<img class="alignnone size-full wp-image-132" title="Workflow Status List Filter" src="/uploads/2009/08/2009-08-25-12-41-53.png" alt="Workflow Status List Filter" width="370" height="128" srcset="/uploads/2009/08/2009-08-25-12-41-53.png 370w, /uploads/2009/08/2009-08-25-12-41-53-300x103.png 300w" sizes="(max-width: 370px) 100vw, 370px" />](/uploads/2009/08/2009-08-25-12-41-53.png)

Listed below are the other values in the enum that you can use for the various states.

```csharp
public enum SPWorkflowStatus
{
    NotStarted = 0,
    FailedOnStart = 1,
    InProgress = 2,
    ErrorOccurred = 3,
    StoppedByUser = 4,
    Completed = 5,
    FailedOnStartRetrying = 6,
    ErrorOccurredRetrying = 7,
    ViewQueryOverflow = 8,
    Max = 15,
}
```