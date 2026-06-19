---
author: Kit
categories:
- SharePoint
date: 2010-11-10T22:12:50Z
guid: http://kitmenke.com/blog/?p=307
id: 307
tags:
- JavaScript
- Prototype
- SharePoint 2007
- SPUtility.js
title: Planned updates for SPUtility.js 0.4
---

SPUtility.js version 0.4 should be getting released soon (week or so??). I wanted to put a quick post out there to highlight some of the upcoming features and fixes.
  

First some fixes:

  * If a field is read only, the label doesn't automatically update.
  
    Update will allows developer to call MakeReadOnly again to update the label. I've been thinking about trying to make the label update automatically when the field value is changed programmatically, but calling MakeReadOnly again should work for now.
  * New separate number field class that will always return a number (fixes issue with returning strings from a number field).

I'll be including some new features as well:

  * Support for Yes/No fields. New SPBooleanField class for allows for getting and setting yes/no fields. SetValue takes true or false and GetValue will return true/false.

Some TODOs:

  * Support for Rich Text Fields
  * Support for lookup fields
  * Getting a SPField by internal column name

I also need to update the SPUtility.js documentation to include information about some properties that I've found to be very useful. I wasn't sure entirely sure at first if they would come in handy. What am I talking about? Let's say we have a requirement from our user to populate a Date field with today's date IF they choose a Status of &#8220;Requested&#8221; from the Status single select dropdown. This is easy with SPUtility.js!

```javascript
var spfStatus = SPUtility.GetSPField('Status');
 Event.observe(spfStatus.Dropdown, 'change', function(event) {
   if ('Requested' == spfStatus.Dropdown.getValue()) {
     var today = new Date();
     SPUtility.GetSPField('Requested Date').SetValue(today.getFullYear(), today.getMonth()+1, today.getDate());
   }
 });
```

Using the <code class="codecolorer javascript default">&lt;span class="javascript">SPChoiceField.&lt;span class="me1">Dropdown&lt;/span>&lt;/span></code> property, we can get the control and observe the onChange event.

I hope to get these features documented with the 0.4 release! As always, any feedback is appreciated; leave me a comment or contact me via Codeplex.