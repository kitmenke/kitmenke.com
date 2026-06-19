---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-10-30T11:06:40Z
guid: http://kitmenke.com/blog/?p=148
id: 148
tags:
- SharePoint 2007
title: Updating choice fields via SharePoint Web Services
---

**May 20, 2010 Update
  
** This post is about using a C# console app to update choice fields via SharePoint web services.  If you are looking for more information on how SharePoint web services can be called via JavaScript, check out my post on [building a simple SharePoint AJAX app](http://kitmenke.com/blog/2010/03/14/a-simple-ajax-app-using-sharepoint-web-services/).

* * *

My problem started with a simple change to a site column. I had originally created a Country field with the following values:

  * US
  * Canada

After a change in requirements, the new values for the field were:

  * Canada
  * Germany
  * Great Britain
  * Ireland
  * United States

After updating the site column, I then needed to update the hundreds of items in the list to use &#8220;United States&#8221; instead of &#8220;US&#8221;. Instead of updating hundreds of items manually, I decided to write a small web services console app. (I was also not able to update it in datasheet... due to content approval maybe???)

However, I quickly ran into problems when attempting to update the multi-select choice field.

## The Setup:

My site column was defined with each choice for Country in alphabetical order:
  
[<img title="2009-10-30 10 24 59" src="/uploads/2009/10/2009-10-30-10-24-59.png" alt="2009-10-30 10 24 59" width="281" height="366" />](/uploads/2009/10/2009-10-30-10-24-59.png)

## Symptoms:

The symptoms of my problem were very strange. After running my app successfully, it seemed as if my updates were simply ignored by the server.

  1. The item is **successfully** updated using the Lists.asmx (it even creates a new version!)
  2. The item does not show the updated value for your multi-select choice field in the UI.

## The Code:

Here is the very simple code that is creating the update message:

```csharp
private static void AddCountryUpdate(ref StringBuilder sb, int count, string id, string newCountry)
 {
     sb.AppendFormat("", count);
     sb.AppendFormat("{0}", id);
     sb.AppendFormat("{0}", newCountry);
     sb.Append("");
 }
```

Calling the following code to update:

```csharp
XmlDocument xmlDoc = new System.Xml.XmlDocument();
 System.Xml.XmlElement elBatch = xmlDoc.CreateElement("Batch");
 elBatch.InnerXml = sb.ToString();
 XmlNode ndReturn = list.ListWebSvc.UpdateListItems(listName, elBatch);
```

## The Results:

Attempting to update Country to be &#8220;;#United States;#Canada;#&#8221; resulted in a &#8220;success&#8221; response (error code of 0x00000000) even when the update was NOT successful:

```xml
 ID="373,Update">
   >0x00000000>
    ows_ID="1311" ows_Country=";#United States;#Canada;#" ows_Modified="2009-10-29 16:01:05" ows__UIVersionString="6.0" />
 >
```

Viewing this in the UI still shows the old value:

[<img class="alignnone size-full wp-image-152" title="Country - Old Value" src="/uploads/2009/10/2009-10-30-09-03-47.png" alt="Country - Old Value" width="300" height="30" />](/uploads/2009/10/2009-10-30-09-03-47.png)

However, only after the values are in order, &#8220;;#Canada;#United States;#&#8221;, will the update actually happen.

```xml
 ID="223,Update">
   >0x00000000>
    ows_ID="1311" ows_Country=";#Canada;#United States;#" ows_Modified="2009-10-30 08:51:45" ows__UIVersionString="7.0" />
 >
```

Viewing this in the UI shows the updated value:

[<img class="alignnone size-full wp-image-153" title="Country - Correct" src="/uploads/2009/10/2009-10-30-09-04-02.png" alt="Country - Correct" width="350" height="30" srcset="/uploads/2009/10/2009-10-30-09-04-02.png 350w, /uploads/2009/10/2009-10-30-09-04-02-300x25.png 300w" sizes="(max-width: 350px) 100vw, 350px" />](/uploads/2009/10/2009-10-30-09-04-02.png)

Notice it still created a version 6.0 and 7.0:

[<img class="alignnone size-full wp-image-161" title="Version History" src="/uploads/2009/10/2009-10-30-11-01-24.png" alt="Version History" width="295" height="201" />](/uploads/2009/10/2009-10-30-11-01-24.png)

## Conclusion:

When updating a multi-select choice field using SharePoint's Lists.asmx web service, **<span style="color: #ff0000;">the order of the choices in the update message are important</span>**. If they are out of order, it will not update.