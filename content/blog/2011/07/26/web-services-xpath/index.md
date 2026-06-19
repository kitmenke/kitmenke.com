---
author: Kit
categories:
- SharePoint
date: 2011-07-26T09:33:18Z
guid: http://kitmenke.com/blog/?p=382
id: 382
tags:
- Example
- SharePoint 2007
- Web Services
- XML
title: Web services, XmlNode, and XPATH
---

I was writing a little web services app to help delete individual permissions from a site and thought about trying XPATH on the result to get what I needed. I had some trouble so thought I'd put together a small example for later reference.

The XML from the Permissions [GetPermissionCollection](http://msdn.microsoft.com/en-us/library/dd586703%28v=office.11%29.aspx) web service looks something like:

```xml
 xmlns="http://schemas.microsoft.com/sharepoint/soap/directory/">
 >
  MemberID="3" Mask="-1" MemberIsUser="False" MemberGlobal="True" GroupName="IT Projects Owners" />
  MemberID="4" Mask="138612833" MemberIsUser="False" MemberGlobal="True" GroupName="IT Projects Visitors" />
  MemberID="5" Mask="1011028719" MemberIsUser="False" MemberGlobal="True" GroupName="IT Projects Members" />
  MemberID="1073741823" Mask="-1" MemberIsUser="True" MemberGlobal="False" UserLogin="SHAREPOINT\system" />
 >
 >
```

The main heartache I had when trying to use XPATH was that the XML uses a **default namespace** (the xmlns=&#8221;&#8221; part). In order to select the Permission nodes, you need to use a XmlNamespaceManager:

```csharp
Permissions webSvc = new Permissions(site.Url.ToString());
 XmlNode result = webSvc.GetPermissionCollection(site.Name, "Web");
 XmlNamespaceManager nsmgr = new XmlNamespaceManager(result.OwnerDocument.NameTable);
 nsmgr.AddNamespace("sp", "http://schemas.microsoft.com/sharepoint/soap/directory/");
 string xpathQuery = "sp:Permissions/sp:Permission";
 XmlNodeList nodeList = result.SelectNodes(xpathQuery, nsmgr);
```

Here is [another example on Stackoverflow](http://stackoverflow.com/questions/4245678/get-sharepoint-list-visible-columns-names-through-web-service-using-c/4249198#4249198).