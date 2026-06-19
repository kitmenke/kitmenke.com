---
author: Kit
categories:
- SharePoint
date: 2010-07-08T13:55:58Z
guid: http://kitmenke.com/blog/?p=258
id: 258
tags:
- JavaScript
- SharePoint 2007
title: Wrap ListViewWebPart Column Headers
---

I recently got asked if it was possible wrap the column headers in a list view web part. The user had a WebPart with quite a few columns with long names and was trying to prevent the page from scrolling left to right.
  

[<img class="alignnone size-full wp-image-259" title="Column Header (nowrap)" src="/uploads/2010/07/2010-07-08-13-48-02.png" alt="Column Header (nowrap)" width="600" height="165" srcset="/uploads/2010/07/2010-07-08-13-48-02.png 600w, /uploads/2010/07/2010-07-08-13-48-02-300x82.png 300w" sizes="(max-width: 600px) 100vw, 600px" />](/uploads/2010/07/2010-07-08-13-48-02.png)

What we needed to do was wrap the column header so that it fit more to the data in the grid.

[<img class="alignnone size-full wp-image-260" title="Column Header (wrapped)" src="/uploads/2010/07/2010-07-08-13-47-27.png" alt="Column Header (wrapped)" width="600" height="165" srcset="/uploads/2010/07/2010-07-08-13-47-27.png 600w, /uploads/2010/07/2010-07-08-13-47-27-300x82.png 300w" sizes="(max-width: 600px) 100vw, 600px" />](/uploads/2010/07/2010-07-08-13-47-27.png)

The solution is relatively simple to implement; all you need is a Content Editor Webpart at the bottom of the page somewhere with the following JavaScript in it:

```javascript
<script type="text/javascript"> 
 function WrapColumnHeaderText(columnName) {
     try {
         var tables = document.getElementsByTagName("table");
         for (i = ; i < tables.length; i++) {
             // find the table that is for the column we're looking for
             var attrItem = tables[i].attributes.getNamedItem("displayname")
             if (null != attrItem && attrItem.nodeValue === columnName) {
                 var cells = tables[i].getElementsByTagName("td");
                 for (c = ; c < cells.length; c++) {
                     if (null != cells[c].attributes.getNamedItem("nowrap")) {
                         cells[c].attributes.removeNamedItem("nowrap");
                         // after removing nowrap, IE won't actually wrap the content
                         // until it "changes".. so we touch it to force it to wrap
                         cells[c].innerHTML = cells[c].innerHTML;
                         break;
                     }
                 }
             }
         }
     } catch (ex) {
         alert(ex.toString());
     }
 }
 
 function WrapColumnHeaders() {
     // TODO: repeat the line below to wrap more columns
     WrapColumnHeaderText("Company Product Code");
 }
 
 _spBodyOnLoadFunctionNames.push("WrapColumnHeaders");
 script>
```