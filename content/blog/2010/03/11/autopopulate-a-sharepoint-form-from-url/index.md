---
author: Kit
categories:
- SharePoint
date: 2010-03-11T12:28:02Z
guid: http://kitmenke.com/blog/?p=237
id: 237
tags:
- JavaScript
- Prototype
- SharePoint 2007
title: Autopopulate a SharePoint Form from URL
---

**November 10, 2010 update:** I've posted the sequel to this post: <a title="Permanent Link to Autopopulate a SharePoint Form from URL (with SPUtility.js)" href="/blog/2010/11/10/autopopulate-a-sharepoint-form-from-url-with-sputility-js/" rel="bookmark">Autopopulate a SharePoint Form from URL (with SPUtility.js)</a>. I highly recommend checking out the new and improved way using [SPUtility.js](http://sputility.codeplex.com/)!

* * *

This post builds off of [my previous post](/blog/2009/06/02/make-a-text-field-read-only-on-editformaspx-without-spd/) about making fields read only. In this case, I needed to autopopulate the NewForm.aspx with a value that gets passed to the form via URL parameters.

In this case, the URL that gets passed in is something like: `/Lists/ProjectTasks/NewForm.aspx?projectID=523`

I wanted to take the `projectID` URL parameter and autopopulate my Project ID field with this value.

[<img class="alignnone size-full wp-image-238" title="Project ID field" src="/uploads/2010/03/projectid.png" alt="" width="258" height="32" />](/uploads/2010/03/projectid.png)

Additionally, we will make this field Read Only so that they can't change the value. Lastly, do this all without SharePoint designer or C# code.

Stick the following code in a Content Editor on your NewForm.aspx or EditForm.aspx. (Remember you can get to edit mode by appending `ToolPaneView=2` on the end, like NewForm.aspx?ToolPaneView=2)

```html
<script type="text/javascript">
function CustomExecuteFunction() {
  AutopopulateInputFromURL('Project ID','projectID');
}

function AutopopulateInputFromURL(title,queryParamName) {
  var inputs = $$('input[title="' + title + '"]');
  if (null != inputs && inputs.length == 1) {
    var input = inputs[];
    var queryParams = location.href.toQueryParams();
    if (queryParams != null && queryParams[queryParamName] != null) {
      input.setValue(queryParams[queryParamName]);
      var label = "" + input.getValue() + "";
      Element.insert(input, {before: label});
      input.hide();
    }
  }
}

_spBodyOnLoadFunctionNames.push("CustomExecuteFunction");
</script>
```

Depends on [Prototype.js](http://www.prototypejs.org/).