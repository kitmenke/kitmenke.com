---
author: Kit
categories:
- SharePoint
date: 2011-01-15T16:15:34Z
guid: http://kitmenke.com/blog/?p=348
id: 348
tags:
- JavaScript
- SharePoint 2007
- SPUtility.js
title: SPUtility 0.5 Released!
---

Version 0.5 of SPUtility.js has been released onto the [codeplex site](http://sputility.codeplex.com/)! Go check it out!

From the [changelog](http://sputility.codeplex.com/wikipage?title=Changelog&referringTitle=Home), here are some of the updates with more detail:

  1. Bug: SPNumberField GetValue() function now disregards commas
  
    If the field contained a value like &#8220;1,000&#8221; the value wouldn't return as a number correctly. GetValue will now automatically convert the value from a string type into a number (ignores other characters).
  2. _Breaking change_: GetSPFields will now return a hashtable instead of the array (see next item)
  3. GetSPField now loads fields into a hashtable instead of an array (dramatically increases speed for forms with a large number of fields)
  
    This is related to how SPUtility stores the fields internally. Instead of using an array, it now uses a hashtable which will significantly speed up GetSPField calls... especially if you have a lot of fields in a list!
  4. GetSPFieldType now attempts to prevent throwing an error when unable to get the field's type (allows for easier Firebugging).
  
    This was more of an annoyance than anything.. made it harder to trap errors in Firebug because this method was throwing an error if it couldn't parse the type.
  5. New MakeEditable function, will undo MakeReadOnly
  
    Very useful if you want to only allow a field to be edited in certain circumstances.
  6. Support for currency fields. MakeReadOnly will display the value with dollar sign, and commas. Uses formatMoney function made by Jonas Raoni Soares Silva.
  
    This leverages a new SPCurrencyField class. Like SPNumberField, GetValue now returns a number. I also developed a Format method but I'm not sure if this is something I want to include in the scope of SPUtility.js. Basically this function enforces a pretty field value... for example: $1,250.95 intead of 1250.95.
  7. Support for single select lookup fields
  
    Lookup fields are strange because the field seems to switch modes depending on the number of items in the lookup list. For less than twenty items, a dropdown is used. For twenty or more, an autocomplete is used. It should work for both but I still have more work to do on these in order to make this class bulletproof.
  8. Reformatted code to meet higher JSLint standards
  
    Mostly just whitespace and parentheses but I also refactored functions to have only one var. This helped because I was using var inside a couple loops.

Lots of good stuff in this release!