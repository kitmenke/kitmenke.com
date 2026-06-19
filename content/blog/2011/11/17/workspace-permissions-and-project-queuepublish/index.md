---
author: Kit
categories:
- SharePoint
date: 2011-11-17T16:04:59Z
guid: http://kitmenke.com/blog/?p=402
id: 402
tags:
- Project Server
- SharePoint 2007
title: Workspace permissions and Project.QueuePublish
---

Imagine you work for a large company that uses Microsoft Project Server. Since you have many employees, having MSP sync individual permissions for every user to every site is simply not scalable. Instead, you'd like to use some Active Directory groups that are already set up to manage permissions in the workspace.

So you disable the workspace permissions sync in PWA:

  1. Navigate to PWA
  2. Click Server Settings
  3. Click Project Workspace Provisioning Settings
  4. Validate that the &#8220;Workspace Permissions&#8221; checkbox is unchecked

<p style="text-align: center;">
  <a href="/uploads/2011/11/msp-workspace-permissions.png"><img class="size-medium wp-image-404 aligncenter" title="MSP Workspace Permissions" src="/uploads/2011/11/msp-workspace-permissions-300x67.png" alt="Microsoft Project Server Workspace Permissions" width="300" height="67" srcset="/uploads/2011/11/msp-workspace-permissions-300x67.png 300w, /uploads/2011/11/msp-workspace-permissions.png 959w" sizes="(max-width: 300px) 100vw, 300px" /></a>
</p>

Then, perhaps you write some code that is wrapped up in a nice SharePoint feature to manage setting up the permissions yourself. Everything works great....

... until you start getting reports that some user's permissions aren't quite right for some project sites.

You investigate and find a whole bunch of permissions with Project Server roles:

  * Readers (Microsoft Office Project Server)
  * Team members (Microsoft Office Project Server)
  * Project Managers (Microsoft Office Project Server)
  * Web Administrators (Microsoft Office Project Server)

More investigation into the PWA Manage Queue Jobs page, you also see &#8220;WSS Workspace Create&#8221; coming up in the queue.

Wait... I thought we disabled the permissions sync! Why is it still syncing from MSP?

If you've read this far, I'm guessing you've also written some custom code for Project Server using the Project Server Interface (PSI) (or you're using the ProjTool app to publish the project). Which means, you're probably using the <a href="http://msdn.microsoft.com/en-us/library/gg177147.aspx" target="_blank">Project.QueuePublish</a> method and you probably have something like this:

```csharp
const string WSS_URL = null; // do not create a project site
 const bool FULL_PUBLISH = true;
 jobId = Guid.NewGuid();
 projectSvc.QueuePublish(jobId, projectId, FULL_PUBLISH, WSS_URL);
```

When fullPublish is true, **<span style="color: #ff0000;"><span style="color: #000000;">it will also sync permissions from MSP</span> even when it is disabled in workspace provisioning settings</span>**. If the project site exists already, it will also sync permissions to that existing site.

The workaround is simple.. set fullPublish to false:

```csharp
const string WSS_URL = null; // do not create a project site
 const bool FULL_PUBLISH = false; // if true, causes the permissions sync
 jobId = Guid.NewGuid();
 projectSvc.QueuePublish(jobId, projectId, FULL_PUBLISH, WSS_URL);
```

TODO: What is the difference when fullPublish is true vs false?

P.S. If you are managing your own security, I recommend using the SharePoint API to create workspaces instead of PSI methods. After you create the workspace from your project template, you can just link it up to the Project in MSP:

```csharp
_project.UpdateProjectWorkspaceAddress(projectUid, newWebName, wssWebAppUid);
```