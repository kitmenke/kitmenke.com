---
author: Kit
categories:
- JavaScript
- SharePoint
date: 2013-05-06T16:22:17Z
guid: http://kitmenke.com/blog/?p=588
id: 588
tags:
- Enterprise Search
- JavaScript
- jQuery
- jsrender
- SharePoint 2007
- Web Services
title: Using JsRender to create SOAP Envelopes
---

I am working on a small query tool that will be used to make AJAX calls against SharePoint's [Enterprise Search Query Web Service](http://msdn.microsoft.com/en-us/library/ms543175(v=office.12).aspx). Previously, I created the SOAP messages by appending a gigantic series of strings together. In this iteration, I wanted to use [JsRender](https://github.com/BorisMoore/jsrender) (my template engine of choice right now and the successor to jQuery templates).

The first template, soapQueryEx, is fairly easy because you can just copy it from the method's page: `/_vti_bin/Search.asmx?op=QueryEx`. The second template is more difficult.. you'll need to craft your &#8220;QueryPacket&#8221; using [the schema](http://msdn.microsoft.com/en-us/library/ms563775(v=office.12).aspx) (or see [Anita's blog post](http://www.itidea.nl/index.php/example-of-using-the-spservices-search-web-service/)).

Embedded below is the code needed to make the AJAX call:

```javascript
function encodeHTML(str) {
  return str.replace(/&/g,'&amp;').replace(/&lt;/g,'&lt;').replace(/&gt;/g,'&gt;').replace(/"/g,'&quot;').replace(/'/g,'&apos;');
}

var queryXml = $('#tmplQueryXml').render({
  'QueryText': $('#txtQueryText').val(),
  'Count': 100,
  'StartAt': 1
});

var soapMessage = $('#soapQueryEx').render({
  'queryXml': encodeHTML($.trim(queryXml))
});

// IE has whitespace after rendering for some reason
// will just keep the trim and change my templates so they look nice
soapMessage = $.trim(soapMessage);

// make the AJAX call to the Search service
$.ajax({ 
  'url': '/_vti_bin/search.asmx',
  'contentType': 'text/xml; charset=utf-8',
  'type': 'POST',
  'data': soapMessage,
  'success': function(data, textStatus, jqXHR) {
    // do stuff with data
    // maybe $(data).find('RelevantResults') ?
  },
  'error': $.proxy(ajaxError, self)
});
```

And the templates:
  
```html
<script type="text/x-jsrender" id="soapQueryEx">
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <QueryEx xmlns="http://microsoft.com/webservices/OfficeServer/QueryService">
      <queryXml>{{:queryXml}}</queryXml>
    </QueryEx>
  </soap:Body>
</soap:Envelope>
</script>

<script type="text/x-jsrender" id="tmplQueryXml">
<QueryPacket xmlns="urn:Microsoft.Search.Query">
  <Query domain="QDomain">
    <SupportedFormats>
      <Format>urn:Microsoft.Search.Response.Document.Document</Format>
    </SupportedFormats>
    <Context>
      <QueryText language="en-US" type="MSSQLFT"><![CDATA[{{:QueryText}}]]></QueryText>
    </Context>
    <Range>
      <Count>{{:Count}}</Count>
      <StartAt>{{:StartAt}}</StartAt>
    </Range>
    <TrimDuplicates>false</TrimDuplicates>
  </Query>
</QueryPacket>
</script>
```