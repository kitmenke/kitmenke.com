---
aktt_notify_twitter:
- false
aktt_tweeted:
- 1
author: Kit
categories:
- SharePoint
date: 2009-06-08T17:37:39Z
excerpt: The SPWeb.OpenWeb() no argument constructor is very useful when it is used
  within a WebPart. However, if used in a console application, it cause some unexpected
  behavior for URLs that do not exist.
guid: http://kitmenke.com/blog/?p=61
id: 61
tags:
- SharePoint 2007
title: The Danger in using SPSite.OpenWeb()
---

Recently I ran into a &#8220;feature&#8221; of SharePoint's SPSite.OpenWeb() method (the no argument constructor specifically). If the OpenWeb() method is used with a URL that is not known to exist, it can result in some unexpected behavior.

Assuming you have the following site structure...

  * SiteCollection 
      * Subsite1
      * Subsite2
      * ...

Can you find the bug?

<div class="codecolorer-container csharp default" style="overflow:auto;white-space:nowrap;width:45em;">
  <table cellspacing="0" cellpadding="0">
    <tr>
      <td class="line-numbers">
        <div>
          1<br />2<br />3<br />4<br />5<br />6<br />7<br />8<br />9<br />10<br />11<br />12<br />
        </div>
      </td>
      
      <td>
        <div class="csharp codecolorer">
          <span class="co1">// !!! warning !!!</span><br /> <span class="co1">// !!! dangerous code ahead !!!</span><br /> <span class="kw4">string</span> server <span class="sy0">=</span> <span class="st0">"http://myserver"</span><span class="sy0">;</span><br /> <span class="kw4">string</span> siteUrl <span class="sy0">=</span> <span class="st0">"/sites/SiteCollection/This Subsite Does Not Exist"</span><span class="sy0">;</span><br /> <span class="kw1">using</span> <span class="br0">&#40;</span>SPSite site <span class="sy0">=</span> <span class="kw3">new</span> SPSite<span class="br0">&#40;</span>server <span class="sy0">+</span> siteUrl<span class="br0">&#41;</span><span class="br0">&#41;</span><br /> <span class="br0">&#123;</span><br /> <span class="xtra ln-xtra">    <span class="kw1">using</span> <span class="br0">&#40;</span>SPWeb web <span class="sy0">=</span> site<span class="sy0">.</span><span class="me1">OpenWeb</span><span class="br0">&#40;</span><span class="br0">&#41;</span><span class="br0">&#41;</span><br /></span>    <span class="br0">&#123;</span><br />         Console<span class="sy0">.</span><span class="me1">WriteLine</span><span class="br0">&#40;</span>site<span class="sy0">.</span><span class="me1">Url</span><span class="br0">&#41;</span><span class="sy0">;</span><br />         Console<span class="sy0">.</span><span class="me1">WriteLine</span><span class="br0">&#40;</span>web<span class="sy0">.</span><span class="me1">Url</span><span class="br0">&#41;</span><span class="sy0">;</span><br />     <span class="br0">&#125;</span><br /> <span class="br0">&#125;</span>
        </div>
      </td>
    </tr>
  </table>
</div>

The problem reveals itself when the URL's are printed. Output:

```
http://myserver/sites/SiteCollection
 http://myserver/sites/SiteCollection
```

The URL for the SPWeb that was just opened is the same as the SPSite's URL: http://myserver/sites/SiteCollection. **There is no exception thrown.**

The behavior I expected, would be some sort of exception when opening a web that does not exist. Even worse is the fact that no error is thrown; it simply defaults to the top level site collection that exists. This means, you get the **<span style="color: #ff0000;">wrong</span>** SPWeb object. This error can happen when you use the no argument OpenWeb method call.

A better way to get an SPWeb object passes an argument to [OpenWeb](http://msdn.microsoft.com/en-us/library/ms474633%28v=office.12%29.aspx) (updated 7/13/09 per Rikard's comment):

<div class="codecolorer-container csharp default" style="overflow:auto;white-space:nowrap;width:45em;">
  <table cellspacing="0" cellpadding="0">
    <tr>
      <td class="line-numbers">
        <div>
          1<br />2<br />3<br />4<br />5<br />6<br />7<br />8<br />9<br />10<br />11<br />12<br />13<br />14<br />15<br />16<br />
        </div>
      </td>
      
      <td>
        <div class="csharp codecolorer">
          <span class="co1">// better code to open a web</span><br /> <span class="kw4">string</span> server <span class="sy0">=</span> <span class="st0">"http://myserver"</span><span class="sy0">;</span><br /> <span class="kw4">string</span> siteUrl <span class="sy0">=</span> <span class="st0">"/sites/SiteCollection"</span><span class="sy0">;</span><br /> <span class="kw4">string</span> subSite <span class="sy0">=</span> <span class="st0">"This Subsite Does Not Exist"</span><span class="sy0">;</span><br /> <span class="kw1">using</span> <span class="br0">&#40;</span>SPSite site <span class="sy0">=</span> <span class="kw3">new</span> SPSite<span class="br0">&#40;</span>server <span class="sy0">+</span> siteUrl<span class="br0">&#41;</span><span class="br0">&#41;</span><br /> <span class="br0">&#123;</span><br />     <span class="kw1">using</span> <span class="br0">&#40;</span>SPWeb web <span class="sy0">=</span> site<span class="sy0">.</span><span class="me1">OpenWeb</span><span class="br0">&#40;</span>subSite<span class="br0">&#41;</span><span class="br0">&#41;</span><br />     <span class="br0">&#123;</span><br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="kw1">if</span> <span class="br0">&#40;</span>web<span class="sy0">.</span><span class="me1">Exists</span><span class="br0">&#41;</span><br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="br0">&#123;</span><br /> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="co1">// do work with the web...</span><br />         &nbsp; &nbsp; Console<span class="sy0">.</span><span class="me1">WriteLine</span><span class="br0">&#40;</span>site<span class="sy0">.</span><span class="me1">Url</span><span class="br0">&#41;</span><span class="sy0">;</span><br />         &nbsp; &nbsp; Console<span class="sy0">.</span><span class="me1">WriteLine</span><span class="br0">&#40;</span>web<span class="sy0">.</span><span class="me1">Url</span><span class="br0">&#41;</span><span class="sy0">;</span><br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="br0">&#125;</span><br />     <span class="br0">&#125;</span><br /> <span class="br0">&#125;</span>
        </div>
      </td>
    </tr>
  </table>
</div>

This code should be able to safely open a web at any location.  Keep in mind, trying to access any of the properties of a web that does not exist will result in:

> System.IO.FileNotFoundException: There is no Web named &#8220;/sites/SiteCollection&#8221;.

Again, the exception is not thrown in the constructor, but when attempting to access the SPWeb's properties... such as web.Url.