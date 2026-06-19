---
aktt_notify_twitter:
- true
aktt_tweeted:
- 1
author: Kit
categories:
- Uncategorized
date: 2009-07-31T09:58:09Z
guid: http://kitmenke.com/blog/?p=103
id: 103
tags:
- AutoIt3
- Example
title: AutoIt3 script to auto resize windows
---

One thing that bugs me all the time is when a window doesn't remember the position that it was resized or moved to. Although it is more of a nuisance than anything else, I decided to write a little AutoIt3 script to automatically resize my Remote Desktop windows to be the certain place and size that I prefer them to be in.

The following script will find any windows that have &#8220;Remote Desktop&#8221; in the title and resize them to be their maximum height and width. In fact, I tweaked it a little bit so that the window actually falls outside of the screen. This way I have more screen real estate for the desktop that I'm working in and I will also be able to quickly move between windows on my desktop.

```
********************************************************************************
; Filename: maximize_window.au3
; Author: Kit Menke
; Date: July 30, 2009
; Description:
;   Resizes any window matching a certain search string to the given dimensions.
;   Useful for Windows you find you have to constantly resize.
;********************************************************************************

; any window that contains the following string will be resized
$windowTitleContains = "Remote Desktop"

; only match exact title names in this script
Opt("WinTitleMatchMode", 3)     ;1=start, 2=subStr, 3=exact, 4=advanced, -1 to -4=Nocase

; the new dimensions of the window to be resized to
; top left corner (x,y) coordinates
$winX = -4
$winY = -29
; the window's new width and height
$winWidth = 1024 - $winX*2
$winHeight = 768 - $winY

; get the list of windows and the string you want to check for
$var = WinList()

For $i = 1 to $var[0][0]
  ; Only display visble windows that have a title matching our search string
  $windowTitle = $var[$i][0]
  $containsTitle = StringInStr($windowTitle,$windowTitleContains)
  If $windowTitle &lt;&gt; "" AND IsVisible($windowTitle) AND 0 &lt;&gt; $containsTitle Then
    ResizeWindowWithExactTitle($windowTitle)
  EndIf
Next

Exit

Func IsVisible($handle)
  If BitAnd( WinGetState($handle), 2 ) Then
    Return 1
  Else
    Return 0
  EndIf
EndFunc

Func ResizeWindowWithExactTitle($winTitle)
    If WinExists($winTitle) Then
        WinMove($winTitle, "", $winX, $winY, $winWidth, $winHeight)
    EndIf
EndFunc
```

&nbsp;

I then set up a little CMD file to call my au3 file (assumes AutoIt3.exe is in the same directory as your script, but you can obviously make this be whatever you want):

> AutoIt3.exe maximize_window.au3

And then I have a shortcut to that CMD file on my quicklaunch that I can call whenever I need to resize my windows.