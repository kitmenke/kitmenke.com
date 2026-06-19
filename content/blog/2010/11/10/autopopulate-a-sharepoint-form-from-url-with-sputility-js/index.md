---
author: Kit
categories:
- SharePoint
date: 2010-11-10T21:34:35Z
guid: http://kitmenke.com/blog/?p=297
id: 297
tags:
- JavaScript
- SharePoint 2007
- SPUtility.js
title: Autopopulate a SharePoint Form from URL (with SPUtility.js)
---

[SPUtility.js](http://sputility.codeplex.com/) is a library used to get/set SharePoint's form fields. We can also use it autopopulate a Newform/Editform based on some URL parameters which are passed to the page!

Our page's URL could be something like: `/Lists/ProjectTasks/NewForm.aspx?projectID=523`

I want to take the `projectID` URL parameter and autopopulate my Project ID field with this value. Then, we'll make the field read only.

[<img class="alignnone size-full wp-image-238" title="Project ID field" src="/uploads/2010/03/projectid.png" alt="" width="258" height="32" />](/uploads/2010/03/projectid.png)

Here's the code (_updated 4/2/2014 to use jQuery version of SPUtility.js_):

```html
<script src="/subsite/Files/jquery-1.11.0.min.js"></script>
<script src="/subsite/Files/sputility.min.js"></script>
<script>
// url parsing from http://stackoverflow.com/a/2880929/98933
var urlParams;
(window.onpopstate = function () {
	var match,
		pl     = /\+/g,  // Regex for replacing addition symbol with a space
		search = /([^&=]+)=?([^&]*)/g,
		decode = function (s) { return decodeURIComponent(s.replace(pl, " ")); },
		query  = window.location.search;
	// fix for sharepoint 2013 URLs which use the hash
	if (query === '') {
		query = window.location.hash.substring(window.location.hash.indexOf('?'));
	}
	query = query.substring(1);
	urlParams = {};
	while (match = search.exec(query))
	   urlParams[decode(match[1])] = decode(match[2]);
})();

// wait for the window to load
$(window).load(function () {
	try {
		var urlValue = urlParams['projectID'];
		SPUtility.GetSPField('Project ID').SetValue(urlValue).MakeReadOnly();
	} catch (ex) {
		alert(ex.toString());
	}
});
</script>
```

To install, you'd simply place the JavaScript above into a Content Editor Web Part on your NewForm.aspx or EditForm.aspx. Take a look at the [SPUtility.js Installation](http://sputility.codeplex.com/wikipage?title=Installation&referringTitle=Documentation) page for detailed instructions.