---
aktt_notify_twitter:
- false
author: Kit
categories:
- JavaScript
date: 2009-06-01T10:17:17Z
guid: http://kitmenke.com/blog/?p=45
id: 45
tags:
- Example
- JavaScript
- Prototype
title: Using Prototype's Element.insert
---

Strangely, Prototype's documentation for Element.insert does not have any examples, so I cooked something up quick:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        "http://www.w3.org/TR/html4/loose.dtd">

<html>

<head>

<title>Template</title>

<script type="text/javascript" src="prototype.js"></script>
<script type="text/javascript">

Event.observe(window, 'load', function() {
    // Using Element.insert to insert after, before, top, or bottom
    Element.insert($('para1'), { before: "<p>This is paragraph zero.</p>" });
    Element.insert($('para1'), { after: "<p>This is paragraph two.</p>" });
   
    // notice we have to insert using the content container
    Element.insert($('content'), { top: "<p>This is the first paragraph.</p>" });
    Element.insert($('content'), { bottom: "<p>This is the last paragraph</p>" });
});

</script>

</head>

<body>

<div id="content">
    <p id="para1">This is paragraph one.</p>
    <p id="para3">This is paragraph three.</p>
</div>

</body>
</html>
```