---
author: Kit
categories:
- SharePoint
date: 2009-06-02T19:09:42Z
guid: http://kitmenke.com/blog/?p=49
id: 49
tags:
- JavaScript
- Prototype
- SharePoint 2007
title: Make a Text field read only on EditForm.aspx without SPD
---

**November 10, 2010 Update:** I've posted the sequel to this post: [Autopopulate a SharePoint Form from URL (with SPUtility.js)](/blog/2010/11/10/autopopulate-a-sharepoint-form-from-url-with-sputility-js/). The code below will *not work in SharePoint 2013* so you should use <a href="http://sputility.codeplex.com/">SPUtility.js</a>!

***

After looking at a couple of examples online, most of them require the use of SharePoint Designer to make a field read only (or to hide it). Here is a method to make a field readonly using only [Prototype](http://www.prototypejs.org/) (javascript library) and a Content Editor Web Part

  1. Edit the EditForm.aspx page
  
    If the Edit Page option is missing from the Site Actions menu, use the ToolPaneView=2 URL parameter.
  
    Ex: /EditForm.aspx?ToolPaneView=2
  2. Add a Content Editor Web Part
  3. Add the following code (in this example, &#8220;Question&#8221; is the name of my field):

```html
<script type="text/javascript">
function SetReadOnly()
{
  var inputs = $$('input[title="Question"]');
  if (null != inputs && inputs.length == 1) {
    var input = inputs[];
    var label = "" + input.getValue() + "";
    Element.insert(input, {before: label});
    input.hide();
  }
}
_spBodyOnLoadFunctionNames.push("SetReadOnly");
</script>
```

References:

  * [Read only field in SharePoint EditForm.aspx](http://nishantrana.wordpress.com/2009/01/30/read-only-field-in-sharepoint-editformaspx/)

* * *

Here is an updated version of the code if you want to use it for multiple fields. You can simply call SetTextFieldReadOnly for each text field you want to hide.

```html
<script type="text/javascript">
function SetTextFieldReadOnly(name)
{
  var inputs = $$('input[title="' + name + '"]');
  if (null != inputs && inputs.length == 1) {
    var input = inputs[];
    var label = "" + input.getValue() + "";
    Element.insert(input, {before: label});
    input.hide();
  }
}
function SetReadOnly()
{
  SetTextFieldReadOnly('Title');
  SetTextFieldReadOnly('Question');
}
_spBodyOnLoadFunctionNames.push("SetReadOnly");
</script>
```