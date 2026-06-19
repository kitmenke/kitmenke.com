---
author: Kit
categories:
- SharePoint
date: 2010-12-11T22:56:56Z
guid: http://kitmenke.com/blog/?p=318
id: 318
tags:
- SharePoint 2007
title: Customizing EditForm.aspx in a Publishing site
---

After recently doing a lot of work heavily customizing NewForm.aspx and EditForm.aspx for a SharePoint list, I ran into a major issue after I moved my list into a publishing site. The issue appeared when I added a webpart onto my EditForm.

[<img class="alignnone size-medium wp-image-321" title="normal view" src="/uploads/2010/12/normal-view-300x105.png" alt="" width="300" height="105" srcset="/uploads/2010/12/normal-view-300x105.png 300w, /uploads/2010/12/normal-view.png 771w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2010/12/normal-view.png)[](/uploads/2010/12/normal-view.png)

As soon as another webpart was added, the entire page switched into some sort of weird publishing mode.

[<img class="alignnone size-medium wp-image-323" title="publishing view" src="/uploads/2010/12/publishing-view-300x129.png" alt="" width="300" height="129" srcset="/uploads/2010/12/publishing-view-300x129.png 300w, /uploads/2010/12/publishing-view.png 775w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2010/12/publishing-view.png)[](/uploads/2010/12/publishing-view.png)

This new mode has a couple of changes to the form:

  1. A publishing toolbar now appears at the top
  2. Every field in the List View Web Part has an additional &#8220;form&#8221; label
  3. Additional JavaScript is added to the page that prompts the user to save whenever an attempts to navigate somewhere differently: To save your changes before continuing, click &#8220;OK&#8221;. To continue without saving changes, click &#8220;Cancel&#8221;.

[<img class="alignnone size-medium wp-image-322" title="publishing view red" src="/uploads/2010/12/publishing-view-red-300x129.png" alt="" width="300" height="129" srcset="/uploads/2010/12/publishing-view-red-300x129.png 300w, /uploads/2010/12/publishing-view-red.png 775w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2010/12/publishing-view-red.png)

[<img title="javascript dialog" src="/uploads/2010/12/javascript-dialog-300x66.png" alt="" width="300" height="66" />](/uploads/2010/12/javascript-dialog.png)

In my case, this new mode definitely would not work for what I was trying to do. The labels increase the size of the form dramatically, the publishing toolbar makes it seem as if you are editing the page rather than a list item, and this new mode does NOT appear on NewForm.aspx which means the forms are dramatically inconsistent.

After googling, I found a couple of people who have had [similar](http://social.msdn.microsoft.com/Forums/en/sharepointcustomization/thread/e10ad027-c28a-43ab-8b48-bb02dc3ca782) [issues](http://social.msdn.microsoft.com/Forums/en-CA/sharepointcustomization/thread/f00d37d3-2254-44fa-8f4d-ae7683c645cf). The root cause seems to be the Office SharePoint Server Publishing site feature. Since disabling the feature on the site was not an option, I had to find another way. JavaScript and CSS was a viable option (since I'm customizing the form using these anyway) but I felt like I needed a more bullet proof method.

My method involves using SharePoint Designer to customize EditForm.aspx. (**<span style="color: #ff6600;">Updated January 13th, 2011</span> after ADE's comment below. Thanks a lot!)** 

**Step 1:** Create a new custom EditForm.aspx by right clicking on EditForm.aspx, click &#8220;New From Existing Page&#8221;

 ****

[<img class="alignnone size-medium wp-image-330" title="new from existing page" src="/uploads/2010/12/new-from-existing-page-300x196.png" alt="" width="300" height="196" srcset="/uploads/2010/12/new-from-existing-page-300x196.png 300w, /uploads/2010/12/new-from-existing-page.png 563w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2010/12/new-from-existing-page.png) ****

**Step 2:** Add the SPNavigation line in right before PlaceHolder Main:

```xml
 ContentPlaceHolderId="PlaceHolderLeftNavBar" runat="server"/>
  ContentPlaceHolderId="SPNavigation" runat="server"/>
 ContentPlaceHolderId="PlaceHolderMain" runat="server">
```

**Step 3:** Save your new CustomEdit.aspx into the same directory as EditForm.aspx

**Step 4:** Verify your list now is associated with your new CustomEdit.aspx by right clicking on the list in SharePoint Designer and clicking &#8220;Properties&#8221;.
  
[<img class="alignnone size-medium wp-image-329" title="list properties" src="/uploads/2010/12/list-properties-300x188.png" alt="" width="300" height="188" srcset="/uploads/2010/12/list-properties-300x188.png 300w, /uploads/2010/12/list-properties.png 532w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2010/12/list-properties.png)
  
Your new edit form should be listed under the Supporting Files tab for Edit item form:
  
[<img class="alignnone size-medium wp-image-331" title="list properties box" src="/uploads/2010/12/list-properties-box-300x291.png" alt="" width="300" height="291" srcset="/uploads/2010/12/list-properties-box-300x291.png 300w, /uploads/2010/12/list-properties-box.png 353w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2010/12/list-properties-box.png)