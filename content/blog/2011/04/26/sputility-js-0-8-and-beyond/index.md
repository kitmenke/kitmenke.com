---
author: Kit
categories:
- SharePoint
date: 2011-04-26T19:18:56Z
guid: http://kitmenke.com/blog/?p=361
id: 361
tags:
- JavaScript
- Prototype
- SharePoint 2007
- SPUtility.js
title: SPUtility.js 0.8 and beyond!
---

Quick update to SPUtility.js today! I've added support for rich text fields, unchecking multi-select checkboxes, and choice fields with fill-in values. I also have some thoughts on future SPUtility.js updates.
  

**Support for rich text fields**

This is an update to the SPNoteField class. In order to get this working, I had to do some funky stuff with internal SharePoint JavaScript functions: <code class="codecolorer text default">&lt;span class="text">RTE_GetEditorIFrame&lt;/span></code>, <code class="codecolorer text default">&lt;span class="text">RTE_GetIFrameContents&lt;/span></code>, and <code class="codecolorer text default">&lt;span class="text">RTE_TransferTextAreaContentsToIFrame&lt;/span></code>.

**Support for unchecking multi-select checkboxes**

I think this really was a bug, but I found that there was no way to uncheck one of the choices of a multi-select choice field. I updated the SetValue function to take an optional boolean second parameter. Passing it true will check the choice... false will uncheck.

```javascript
//uncheck the Bravo choice
 SPUtility.GetSPField('Multiselect Column').SetValue('Bravo', false);
```

**Support for choice fields with fill-in values**

I finally buckled down and implemented the code to support choice fields with fill-in values. Turns out this was a little more complicated than I first expected. I'm not sure how this will work with languages other than english because it seems difficult to find the &#8220;Specify your own value&#8221; radio button. It also complicates the SPChoiceField class quite a bit.

Either way, I tried to keep things as simple as possible and keep most of the burden in the library rather than on the developer/user:

  1. Detects the presence of the fill-in textbox for every different type of choice field (radio, dropdown, multi-select)
  2. <code class="codecolorer text default">&lt;span class="text">SetValue&lt;/span></code> is smart: if the value passed is a choice, then that choice is set. If not, then the fill-in value is set.
  3. <code class="codecolorer text default">&lt;span class="text">GetValue&lt;/span></code> is also able to detect when the fill-in value is present

I'm hoping to break some of this logic down into smaller classes in the future but this will really all be cleanup work on my end. Which leads me into the next part...

**Thoughts about future updates**

My wishlist for future updates currently is:

  1. Better documentation!
  
    I plan to start using [PDoc](http://pdoc.org/) (what Prototype.js uses for it's [API documentation](http://api.prototypejs.org/))
  2. To support #1, I'd like to break SPUtility.js into more manageable chunks and use [Sprockets](https://github.com/sstephenson/sprockets)
  3. To support #1 and #2, I'd like to start using github for the day to day version control. This is something I debated but I think I can use it for the full source and still use the codeplex site for the releases. I'm not sure where I'll host the documentation yet.

Go download SPUtility.js at <http://sputility.codeplex.com/>. Feel free to post comments with feedback!