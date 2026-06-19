---
author: Kit
categories: 
- SharePoint
date: 2016-02-15T09:47:16Z
guid: http://kitmenke.com/blog/?p=840
id: 840
tags: 
- SSRS
- SharePoint 2007
- SharePoint 2013
title: SSRS Subscribe Action and SharePoint Permissions
---

For SQL Server Reporting Services (SSRS) reports there is the ability to create subscriptions. Subscriptions can be scheduled to run on a certain schedule to send emails, export reports to SharePoint document libraries, or save to windows file shares.

Depending on your requirement, you may need to grant or remove access to the subscribe action and can be managed by creating or editing the default SharePoint roles: Read, Contribute, Full Control, etc.

There are two main ways to create a subscription:

  1. Run the report and click Actions -> Subscribe on the Reporting Services Viewer page (RSViewerPage.aspx).
  2. Find the report in the Document Library and then clicking Manage Subscriptions in the edit menu.

To remove the ability for a user to create subscriptions:

  1. Site Actions
  2. Site Settings
  3. Advanced Permissions
  4. Settings -> Permission Levels
  5. Click on the role, for example &#8220;Read&#8221;
  6. Uncheck the _Create Alerts  &#8211;  Create e-mail alerts._ permission

| Before                                                                                                                                                                      | After                                                                                                                                                                       |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![SSRS Subscribe Before](/uploads/2016/01/ssrssubscribe.png)   | ![SSRS Subscribe After](/uploads/2016/01/ssrssubscribe3.png) |
| ![SSR Subscribe](/uploads/2016/01/ssrssubscribe2.png) | ![SSRS Subscribe](/uploads/2016/01/ssrssubscribe4.png) |

Note: This will also remove their ability to create SharePoint alerts as well (Actions -> Alert me in a list or library).
