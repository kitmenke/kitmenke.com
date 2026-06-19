---
author: Kit
categories:
- SharePoint
date: 2013-10-31T22:07:34Z
guid: http://kitmenke.com/blog/?p=645
id: 645
tags:
- InfoPath 2013
- JavaScript
- SharePoint 2013
title: Pass user information from SharePoint 2013 to InfoPath 2013
---

InfoPath is a common tool used to create forms hosted on SharePoint. Due to the limited development options available for Office 365, I think this places an increased focus on InfoPath when it comes to creating a solution. Unfortunately, I found that it was relatively difficult to get the current user's information to be displayed or saved within an Infopath form.

Typically, within the form you would just [make a web service call to the UserProfileService.asmx in order to retrieve the current user](http://blogs.microsoft.co.il/blogs/itaysk/archive/2007/04/05/InfoPath-_2D00_-Get-the-current-user-without-writing-code.aspx). Unfortunately, I was unable to get this working when the form was deployed to an Office 365 site. [[1](http://community.office365.com/en-us/forums/154/t/185751.aspx)] This [KB article](http://support.microsoft.com/kb/2674193) suggests this is because the Office 365 servers have loopback protection enabled.

My solution involves the following:

  * Office 365 SharePoint 2013 site
  * InfoPath 2013 Form with one  parameter made available to use in a web part connection
  * InfoPath Form Web Part
  * Script Editor Web Part
  * Query String (URL) Filter Web Part

**Step 1:** Create your InfoPath form (or cheat and download the one I've created <a href="/uploads/2013/10/UserInformation.zip" rel="attachment wp-att-649">User Information Infopath form</a>). I created textboxes for each of the User fields as well as one additional one that will be used to pass information into the form.[<img class="alignnone size-full wp-image-648" alt="infopath-form" src="/uploads/2013/10/infopath-form.png" width="1248" height="1040" srcset="/uploads/2013/10/infopath-form.png 1248w, /uploads/2013/10/infopath-form-300x250.png 300w, /uploads/2013/10/infopath-form-1024x853.png 1024w" sizes="(max-width: 1248px) 100vw, 1248px" />](/uploads/2013/10/infopath-form.png)I created rules on the txtUrlParameter field to populate my other fields. This is because the InfoPath form web part can <span style="text-decoration: underline;">only accept one web part connection at a time</span> so we have to cram all our parameters into one URL parameter. I decided to use a format like:

```
0[i:0#.f|membership|myemail@example.com]1[11]2[Kit Menke]3[myemail@example.com]
```

This allows me to keep my rules relatively simple inside the InfoPath form:

[<img class="alignnone size-full wp-image-650" alt="infopath-rule-details" src="/uploads/2013/10/infopath-rule-details.png" width="499" height="301" srcset="/uploads/2013/10/infopath-rule-details.png 499w, /uploads/2013/10/infopath-rule-details-300x180.png 300w" sizes="(max-width: 499px) 100vw, 499px" />](/uploads/2013/10/infopath-rule-details.png)

**Step 2:** Publish the form to your Office 365 site. I created a new Forms library for testing. Make sure the Url Parameter field available to use in a web part connection.

[<img class="alignnone size-full wp-image-646" alt="infopath-publishingwizard-parameters" src="/uploads/2013/10/infopath-publishingwizard-parameters.png" width="917" height="464" srcset="/uploads/2013/10/infopath-publishingwizard-parameters.png 917w, /uploads/2013/10/infopath-publishingwizard-parameters-300x151.png 300w" sizes="(max-width: 917px) 100vw, 917px" />](/uploads/2013/10/infopath-publishingwizard-parameters.png)

**Step 3:** I edited my web part page and added on my three web parts:

  1. Forms -> InfoPath Form Web Part
  2. <span style="line-height: 1.5;">Media and Content -> Query String (URL) Filter Web Part</span>
  3. Filters -> Script Editor Web Part

_Note: The &#8220;Current User Filter&#8221; web part might catch your eye. Unfortunately, this web part passes a user in the format: i:0#.f|membership|myemail@example.com. Not what we need inside an InfoPath form._

**Step 4:** Configure the InfoPath Form Web Part to point to the form you published in step 2.

**Step 5:** Configure the Query String (URL) Filter Web Part to use a URL Parameter called &#8220;userinfo&#8221;.

**Step 6: **Connect it to the InfoPath Form Web Part so that the userinfo parameter will populate the Url Parameter field in your InfoPath form.

**Step 7:** Add the following script inside your Script Editor Web Part (modified the code I found <http://sharepoint.stackexchange.com/a/73037/2070>). The script looks up the current user using SharePoint's Client Object Model and creates a dynamic link.  You would put the Script editor web part on another page (but for demo purposes, it works fine on the same page too). You will want to modify the HREF of the link to point the page you want.

```html
<script type="text/javascript">
function CustomExecuteFunction() {
	var self = this;
	self.context = new SP.ClientContext.get_current();
	self.web = context.get_web();
	self.currentUser = web.get_currentUser();
	context.load(self.currentUser);
	
	self.asyncSuccess = function(sender, args) {
		var user = this.currentUser;
		console.log('asyncSuccess', user);
		document.getElementById('userLoginName').innerHTML = user.get_loginName(); 
		document.getElementById('userId').innerHTML = user.get_id();
		document.getElementById('userTitle').innerHTML = user.get_title();
		document.getElementById('userEmail').innerHTML = user.get_email();
		
		var url = '?userinfo=';
		url += '0[' + encodeURIComponent(user.get_loginName()) + ']';
		url += '1[' + encodeURIComponent(user.get_id()) + ']';
		url += '2[' + encodeURIComponent(user.get_title()) + ']';
		url += '3[' + encodeURIComponent(user.get_email()) + ']';
		document.getElementById('customButton').href += url;
	};
	
	self.asyncFailure = function(sender, args) {
		alert('Request failed. \nError: ' + args.get_message() + '\nStackTrace: ' + args.get_stackTrace());
	};
	
	// actually fires off the AJAX request
	// more info: http://msdn.microsoft.com/en-us/library/dn168907.aspx
	context.executeQueryAsync(
		Function.createDelegate(self,self.asyncSuccess), 
		Function.createDelegate(self,self.asyncSuccess)
	);
}

ExecuteOrDelayUntilScriptLoaded(CustomExecuteFunction,'sp.js');
</script>
<div>Current Logged User:
	<span id="userLoginName"></span>
	<span id="userId"></span>
	<span id="userTitle"></span>
	<span id="userEmail"></span>
	<a id="customButton" href="https://example.sharepoint.com/SitePages/Test.aspx">Click me!</a>
</div>
```

Here is the completed InfoPath web part with pre-populated data from the current user!

[<img class="alignnone size-full wp-image-647" alt="infopath-form-webpart" src="/uploads/2013/10/infopath-form-webpart.png" width="852" height="334" srcset="/uploads/2013/10/infopath-form-webpart.png 852w, /uploads/2013/10/infopath-form-webpart-300x117.png 300w" sizes="(max-width: 852px) 100vw, 852px" />](/uploads/2013/10/infopath-form-webpart.png)

Download the InfoPath form I've created: <a href="/uploads/2013/10/UserInformation.zip" rel="attachment wp-att-649">UserInformation.zip</a>.