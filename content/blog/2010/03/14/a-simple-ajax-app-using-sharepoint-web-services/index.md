---
author: Kit
categories:
- SharePoint
date: 2010-03-14T19:08:18Z
guid: http://kitmenke.com/blog/?p=224
id: 224
tags:
- JavaScript
- Prototype
- SharePoint 2007
title: A Simple AJAX App using SharePoint Web Services
---

In this era of Web 2.0 web applications, I think it was only a matter of time before I needed to investigate how I could apply AJAX to SharePoint. With a little JavaScript and SharePoint's web services, you can build fancy apps that can easily be dropped into a document library. And after reading [Glen Cooper's blog post](http://blog.glenc.net/2007/04/20/calling-sharepoint-web-services-from-javascript/) on the subject, I figured it was something definitely worth investigating.

I also took this opportunity to improve a process I have to do all the time: what are the internal column names in a list? To get the list information, I built a simple AJAX app that consumes the Lists.asmx web service. Please keep in mind that this code will have be uploaded into SharePoint in order to work (a document library will work) and will not run from your desktop.

## Step 0: HTML for user input and displaying results

Since I will be calling the GetList method on Lists.asmx, I will need two parameters from the user: Site URL and List Name. Site URL gives the web service context to a particular web and List Name should be a list within that web. I also specify a DIV which is where I will display the list's information.

```html
<div id="controls">
 <fieldset>
 <legend>List Information</legend>
 <label for="txtSiteURL">Site URL:</label><input id="txtSiteURL" type="text" style="width:400px" value="http://server/sites/SiteCollection/SubSite" /><br/>
 <div>Example: http://server/sites/SiteCollection/SubSite</div>
 <label for="txtListName">List Name:</label><input id="txtListName" type="text" value="Images" /><br/>
 <div>Example: Images</div>
 <input id="btnExecute" type="button" value="Get Columns" />
 </fieldset>
 </div>
 
 <div id="result">
 (results)
 </div>
```

Now that we have our HTML form, we can work on the JavaScript code.

## Step 1: Building a SOAP Message

To communicate with the web service, we need to use SOAP. Luckily, Glen has already created a very handy class for taking care of these operations. It is also reusable so it will come in handy for other web service calls. I tweaked his code a little bit and then put it into SoapUtils.js (also, please note this code depends on [Prototype.JS](http://www.prototypejs.org/)):

```javascript
/*
 SoapUtils.js
 Author: Glen Cooper
 http://blog.glenc.net/2007/04/20/calling-sharepoint-web-services-from-javascript/
 */
 var Soap = {
 createEnvelope: function(action, ns, parameters) {
 var soap = '';
 soap += '';
 soap += '  ';
 soap += '    <' + action + ' xmlns="' + ns + '">';
 soap += Soap.__parseParameters(parameters);
 soap += '     + action + '>';
 soap += '  ';
 soap += '';
 return soap;
 },
 
 __parseParameters: function(parameters) {
 var params = "";
 if (typeof parameters == 'object') {
 // check if we were provided an array or an object
 if (typeof parameters.push == 'function') {
 for (var i = , length = parameters.length; i < length; i += 2) {
 params += "<" + parameters[i] + ">" + parameters[i+1] + " + parameters[i] + ">";
 }
 }
 else {
 $H(parameters).each(function(pair) {
 params += "<" + pair.key + ">" + pair.value + " + pair.key + ">";
 });
 }
 }
 return params;
 }
 }
```

This utility class handles a lot of the grunt work for creating the SOAP message. It makes it pretty easy to grab the input, build the message, and then send it to the server. Next, I need to give it the specifics it needs to build the actual message. This is all contained inside my Execute function:

```javascript
function Execute() {
 $('result').innerHTML = "Executing..."
 
 var txtSiteUrl = $('txtSiteURL').getValue();
 var txtListName = $('txtListName').getValue();
 
 // build parameter object
 var parameters =
 {
 listName: txtListName
 }
 
 // create soap envelope
 // can find the action and namespace from the WSDL page
 // ex: http://servername/_vti_bin/Lists.asmx?op=GetList
 var soap = Soap.createEnvelope(
 "GetList", // action
 "http://schemas.microsoft.com/sharepoint/soap/", // namespace
 parameters);
 
 if (!txtSiteUrl.endsWith('/')) {
 txtSiteUrl += '/';
 }
 txtSiteUrl += '_vti_bin/Lists.asmx';
 
 // call web service
 new Ajax.Request(
 txtSiteUrl,
 {
 method: "post",
 contentType: "text/xml",
 postBody: soap,
 onSuccess: ajaxSuccess,
 onFailure: ajaxError
 });
 }
```

The Execute method is then bound to the &#8220;Get Columns&#8221; button on page load:

```javascript
Event.observe(window,'load',function(){
 Event.observe('btnExecute','click',Execute);
 });
```

## Step 2: Parsing the Web Service's XML response

At this point, we are ready to handle the response from the web service. Unfortunately, parsing XML using JavaScript was not exactly the easiest experience. I haven't found an easy way to do it, so if you have any ideas, please let me know in the comments.

In any case, if an error occurs, I simply want to print the error message:

```javascript
function ajaxError(result) {
 Element.insert($('result'), {bottom:'' + result.responseText + ''});
 }
```

A success message is a lot more complicated since we want to extract information out of the returned XML. In fact, if you look at the entire response from the webservice after calling GetList, the returned result for a simple Images library is something like 1100 lines of XML. The majority of this is taken up by the full description of each column in the list. There is a LOT of junk that we are simply going to be skipping over. For sanity sake, I'm not going to post the entire response, but useful snippets of interest.

Overall, the response will look something like:

```xml
 xmlns="http://schemas.microsoft.com/sharepoint/soap/">
 >
  RequireCheckout="False" EnableMinorVersion="False" ShowUser="True" Ordered="False" MultipleDataList="False" Hidden="False" EnableVersioning="True" EnableModeration="False" EnableAttachments="False" AllowMultiResponses="False" AllowDeletion="True" HasUniqueScopes="False" WorkFlowId="" MajorWithMinorVersionsLimit="0" MajorVersionLimit="0" ScopeId="317f19e2-4ba8-4ecc-82f3-9d846a270840" SendToLocation="" WebId="0bb1435c-d56a-4f6a-93a7-77ad6990a48c" WebFullUrl="/sites/SiteCollection/SubSite1/SubSite2" EmailAlias="" EmailInsertsFolder="" EventSinkData="" EventSinkClass="" EventSinkAssembly="" Author="630" WriteSecurity="1" ReadSecurity="1" RootFolder="/sites/SiteCollection/SubSite1/SubSite2/Images1" AnonymousPermMask="0" ItemCount="6" Flags="4232" WebImageHeight="480" WebImageWidth="640" ThumbnailSize="160" Direction="none" Version="0" LastDeleted="20081001 01:58:47" Modified="20100205 08:40:53" Created="20081001 01:58:47" ServerTemplate="109" FeatureId="00bfea71-52d4-45b3-b544-b1c71b620109" BaseType="1" Name="{2CDA4792-6D4E-4D9F-BC5B-F862D5235061}" ImageUrl="/_layouts/images/itil.gif" Description="" Title="Images" ID="{2CDA4792-6D4E-4D9F-BC5B-F862D5235061}" MobileDefaultViewUrl="" DefaultViewUrl="/sites/SiteCollection/SubSite1/SubSite2/Images1/Forms/AllItems.aspx" DocTemplateUrl="">
 >
 ... all fields are listed here in > nodes
 >
 >
 ...
 >
 >
 ...
 >
 >
 >
 >
```

Notice, the List node contains a LOT of useful information. Once we have that node, we can display a lot of it using this function:

```javascript
function addListInformation(nodeList) {
 var listTitle = nodeList.attributes.getNamedItem('Title').value;
 var listImg = nodeList.attributes.getNamedItem('ImageUrl').value;
 var listDesc = nodeList.attributes.getNamedItem('Description').value;
 var listCount = nodeList.attributes.getNamedItem('ItemCount').value;
 var listViewURL = nodeList.attributes.getNamedItem('DefaultViewUrl').value;
 
 var output = '+ listViewURL + '">+ listImg + '" alt="' + listTitle + '" />'
 output += '+ listViewURL + '">' + listTitle + '';
 output += '# of items = ' + listCount + '';
 output += '' + listDesc + '';
 Element.insert('result', {bottom:output});
 }
```

Also, since we are interested in the internal column names, we want information about each of the fields. All of the columns are listed inside the Fields node (see above). A lot of them have attributes similar to below:

```xml
 FromBaseType="TRUE" StaticName="Created" SourceID="http://schemas.microsoft.com/sharepoint/v3" StorageTZ="TRUE" DisplayName="Created" Name="Created" Type="DateTime" ReadOnly="TRUE" RowOrdinal="0" ColName="tp_Created" ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}">>
  FromBaseType="TRUE" StaticName="Author" SourceID="http://schemas.microsoft.com/sharepoint/v3" DisplayName="Created By" Name="Author" List="UserInfo" Type="User" ReadOnly="TRUE" RowOrdinal="0" ColName="tp_Author" ID="{1df5e554-ec7e-46a6-901d-d85a3881cb18}">>
```

So, for each of the Field nodes, we are interested in the StaticName and DisplayName attributes:

```javascript
function addFieldInformation(nodes) {
 if (nodes.length <= ) {
 Element.insert(container, {bottom:"No results to display."});
 return;
 }
 
 var output = 'Display NameInternal Name';
 for (i = ; i < nodes.length; i++) {
 var attr = nodes[i].attributes;
 // there are a lot more attributes, but right now only care about the Display Name
 // and the internal name
 var displayName = attr.getNamedItem('DisplayName').value;
 var internalName = attr.getNamedItem('StaticName').value;
 output += '' + displayName + '' + internalName + '';
 }
 output += '';
 
 Element.insert('result', {bottom:output});
 }
```

And finally, to bring it all together, we are calling these functions inside our success function:

```javascript
function ajaxSuccess(result) {
 // we're going to be adding HTML so reset this to be empty
 $('result').innerHTML = "";
 
 // put it in xmlDoc to make it shorter
 var xmlDoc = result.responseXML;
 
 // get the List node and show some of the attributes that are interesting
 addListInformation(xmlDoc.getElementsByTagName("List")[]);
 // then select the Fields node and get all of its children (Field nodes)
 addFieldInformation(xmlDoc.getElementsByTagName("Fields")[].childNodes);
 }
```

All together now:

```html
 "http://www.w3.org/TR/html4/loose.dtd">
 
 <html>
 
 <head>
 
 <title>List Information - Kit's Utilities</title>
 
 
 <link href="css/reset.css" rel="stylesheet" type="text/css" />
 <link href="css/utilities.css" rel="stylesheet" type="text/css" />
 
 
 <script type="text/javascript" src="scriptaculous/prototype.js"></script>
 <script type="text/javascript" src="ajax/SoapUtils.js"></script>
 
 <script type="text/javascript">
 Event.observe(window,'load',function(){
 Event.observe('btnExecute','click',Execute);
 });
 
 function Execute() {
 $('result').innerHTML = "Executing..."
 
 var txtSiteUrl = $('txtSiteURL').getValue();
 var txtListName = $('txtListName').getValue();
 
 // build parameter object
 var parameters =
 {
 listName: txtListName
 }
 
 // create soap envelope
 // can find the action and namespace from the WSDL page
 // ex: http://servername/_vti_bin/Lists.asmx?op=GetList
 var soap = Soap.createEnvelope(
 "GetList", // action
 "http://schemas.microsoft.com/sharepoint/soap/", // namespace
 parameters);
 
 if (!txtSiteUrl.endsWith('/')) {
 txtSiteUrl += '/';
 }
 txtSiteUrl += '_vti_bin/Lists.asmx';
 
 // call web service
 new Ajax.Request(
 txtSiteUrl,
 {
 method: "post",
 contentType: "text/xml",
 postBody: soap,
 onSuccess: ajaxSuccess,
 onFailure: ajaxError
 });
 }
 
 function ajaxSuccess(result) {
 // we're going to be adding HTML so reset this to be empty
 $('result').innerHTML = "";
 
 // put it in xmlDoc to make it shorter
 var xmlDoc = result.responseXML;
 
 // get the List node and show some of the attributes that are interesting
 addListInformation(xmlDoc.getElementsByTagName("List")[0]);
 // then select the Fields node and get all of its children (Field nodes)
 addFieldInformation(xmlDoc.getElementsByTagName("Fields")[0].childNodes);
 }
 
 function addListInformation(nodeList) {
 var listTitle = nodeList.attributes.getNamedItem('Title').value;
 var listImg = nodeList.attributes.getNamedItem('ImageUrl').value;
 var listDesc = nodeList.attributes.getNamedItem('Description').value;
 var listCount = nodeList.attributes.getNamedItem('ItemCount').value;
 var listViewURL = nodeList.attributes.getNamedItem('DefaultViewUrl').value;
 
 var output = '<table><tr><td><a href="' + listViewURL + '"><img src="' + listImg + '" alt="' + listTitle + '" /></a></td>'
 output += '<td><a href="' + listViewURL + '">' + listTitle + '</a></td>';
 output += '<td># of items = ' + listCount + '</td>';
 output += '<td>' + listDesc + '</td></div></tr></table>';
 Element.insert('result', {bottom:output});
 }
 
 function addFieldInformation(nodes) {
 if (nodes.length <= ) {
 Element.insert(container, {bottom:"No results to display."});
 return;
 }
 
 var output = 'Display Name</th><th>Internal Name</th></tr></thead><tbody>';
 for (i = 0; i < nodes.length; i++) {
 var attr = nodes[i].attributes;
 // there are a lot more attributes, but right now only care about the Display Name
 // and the internal name
 var displayName = attr.getNamedItem('DisplayName').value;
 var internalName = attr.getNamedItem('StaticName').value;
 output += '' + displayName + '</td><td>' + internalName + '</td></tr>';
 }
 output += '</tbody></table>';
 
 Element.insert('result', {bottom:output});
 }
 
 function ajaxError(result) {
 Element.insert($('result'), {bottom:'<span style="color:red">' + result.responseText + '</span>'});
 }
 
 </script>
 
 </head>
 
 <body>
 
 <div id="container">
 <div id="controls">
 <fieldset>
 <legend>List Information</legend>
 <label for="txtSiteURL">Site URL:</label><input id="txtSiteURL" type="text" style="width:400px" value="http://server/sites/SiteCollection/SubSite" /><br/>
 <div>Example: http://server/sites/SiteCollection/SubSite</div>
 <label for="txtListName">List Name:</label><input id="txtListName" type="text" value="Images" /><br/>
 <div>Example: Images</div>
 <input id="btnExecute" type="button" value="Get Columns" />
 </fieldset>
 </div>
 
 <div id="result">
 (results)
 </div>
 </div>
 
 </body>
 </html>
```