---
author: Kit
categories:
- JavaScript
date: 2013-02-14T10:21:38Z
guid: http://kitmenke.com/blog/?p=545
id: 545
tags:
- jsrender
- knockout
title: Custom Debug JsRender tag
---

I've been learning a lot about JsRender and Knockout.js after finding [Ryan's answer on StackOverflow](http://stackoverflow.com/questions/11077539/knockoutjs-third-party-templating-library-jsrender). I've created a couple of templates and started to use the `{{for}}{{/for}}` tag.

I was having trouble figuring out how to access the parent object's data from within the for loop. After reading John Papa's awesome posts, [Using JsRender with JavaScript and HTML](http://msdn.microsoft.com/en-us/magazine/hh882454.aspx) and [Advanced JsRender Templating Features](http://msdn.microsoft.com/en-us/magazine/hh975379.aspx) I came up with a quick custom &#8220;Debug&#8221; tag to help me understand.

```JavaScript
/**
 * Custom JsRender debug tag
 * Log a message and an object to the console.
 * Usage: {{debug #parent message='Inside the for loop'/}}
 **/
$.views.tags({
    debug: function(obj) {
        var props = this.props;
        // output a default message if the user didn't specify a message
        var msg = props.message || 'Debug:';
        console.log(msg, obj);
    }
});
```

For example, I was trying to figure out what the #parent object looked like so I put the debug tag inside my for loop:

```
{{debug #parent message='Inside the for loop'/}}
```

And got this output to the console:

[<img class="alignnone size-full wp-image-558" alt="Chrome Console JsRender debug tag" src="/uploads/2013/02/chrome-console-log.png" width="484" height="299" srcset="/uploads/2013/02/chrome-console-log.png 484w, /uploads/2013/02/chrome-console-log-300x185.png 300w" sizes="(max-width: 484px) 100vw, 484px" />](/uploads/2013/02/chrome-console-log.png)

Neato!

More links... [someone who was having a similar issue](https://github.com/BorisMoore/jsrender/issues/187) which led to [Boris's example on accessing parent data](http://borismoore.github.com/jsrender/demos/step-by-step/11_accessing-parent-data.html).