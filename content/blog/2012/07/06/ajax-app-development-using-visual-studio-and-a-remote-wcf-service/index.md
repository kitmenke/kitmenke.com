---
author: Kit
categories:
- JavaScript
date: 2012-07-06T08:41:50Z
guid: http://kitmenke.com/blog/?p=459
id: 459
tags:
- ASP.NET
- JavaScript
- jQuery
- Visual Studio
- WCF
title: AJAX app development using Visual Studio and a remote WCF service
---

My team has recently created quite a few awesome WCF services that can surface data via REST or JSON. The next step, was building an interface on top of the services using AJAX.

Since the services were hosted on a SharePoint farm, it is super easy to just upload an HTML file into a document library and start coding away. However, eventually this process breaks down; I needed a way to deploy these files to different environments and keep them in some sort of version control system. (No, a document library doesn't count as version control)

Enter Visual Studio 2008 and the ASP.NET Web Application project template. The built-in development server can serve up everything (HTML/CSS/JS) ... everything except the AJAX  calls to the WCF service which are using relative URLs. For example:

```javascript
$.ajax({ 
  'url': '/_vti_bin/EHI/Users.svc/json/GetCurrentUser',
  'dataType': 'json',
  'type': 'GET',
  'success': function(data) {
     self.currentUser(new Employee(data.d));
  },
  'error': function() {
     console.log("Error getting current user.");
  }
});
```

Locally, all of our AJAX calls will <span style="color: #ff0000;">fail</span>!!! This is because our requests are going to `http://localhost:1919/_vti_bin/EHI/Users.svc/json/GetCurrentUser` and since we aren't re-hosting our WCF service inside our ASP.NET project, the request won't work.

**Bad Idea #1:** Add server name to the beginning of all the URLs!

Cross-domain calls aren't allowed. Plus, I'd have to update the code when deploying to each environment. No good.

**Bad Idea #2:** Create a dummy service hosted in the asp.net project.

I saw this recommended a couple of places across the web. What happens when the remote service gets updated on the server? We have to change our dummy service too? Sounds like a lot of work (especially when you are first creating the code)!

**The solution** I came up with, involves a simple HTTPHandler. All the handler does, is act like a proxy: forward the request to the DEV server where my WCF service is hosted and return the response.

```csharp
public class ProxyHandler : IHttpHandler
{
    const string SERVER_URL = "http://dev-server";

    #region IHttpHandler Members

    public bool IsReusable
    {
        get { return false; }
    }

    public void ProcessRequest(HttpContext context)
    {
        HttpWebRequest webRequest = (HttpWebRequest)WebRequest.Create(SERVER_URL + context.Request.Url.PathAndQuery);
        webRequest.Method = context.Request.HttpMethod;
        webRequest.Credentials = System.Net.CredentialCache.DefaultNetworkCredentials;
        HttpWebResponse response = (HttpWebResponse)webRequest.GetResponse();
        context.Response.StatusCode = (int)response.StatusCode;
        context.Response.Status = string.Format("{0} {1}", (int)response.StatusCode, response.StatusCode);
        context.Response.ContentType = response.ContentType;
        context.Response.Write(new StreamReader(response.GetResponseStream()).ReadToEnd());
    }

    #endregion
}
```

Next, we need a couple of web.config changes to catch any calls to `/_vti_bin/EHI/*`. I added a node to the end of the `<httpHandlers>` section:

```xml
<add verb="*" path="/_vti_bin/EHI/*.*" validate="false" type="YourNamespace.ProxyHandler, YourNamespace"/>
```

The next problem I encountered was my handler would never fire when a request to /\_vti\_bin/EHI/Users.svc was made. Instead, I was getting a 404 error. Since I'm not actually hosting my .svc service locally in my project, I don't want it to look there for a .svc file... in fact I don't want it to interpret .svc files at all.

As always, StackOverflow had the answer:  [How can I override a .svc file in my routing table?](http://stackoverflow.com/q/3662555/98933).  The change, is to the `<compilation debug="true">` section:

```xml
<buildProviders>
  <remove extension=".svc"/>
</buildProviders>
```

That's it! Now we have:

  1. A nice, neat visual studio solution saved in version control
  2. My HTML/JS code is clean with relative URLs so that when I deploy this to each environment no changes are needed
  3. With one config change, I can point my server at any environment where the WCF service is hosted to test