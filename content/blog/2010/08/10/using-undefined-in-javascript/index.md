---
author: Kit
categories:
- JavaScript
date: 2010-08-10T12:37:52Z
guid: http://kitmenke.com/blog/?p=269
id: 269
tags:
- Example
- JavaScript
- Links
title: Using undefined in JavaScript
---

Working with JavaScript can definitely be painful, but I always love stumbling across interesting and strange features of the language. I just found [wtfjs.com](http://wtfjs.com/) which led me to these [two](http://stackoverflow.com/questions/1995113/strangest-language-feature/2008728#2008728) [answers](http://stackoverflow.com/questions/1995113/strangest-language-feature/2010740#2010740) on stackoverflow.

[The first answer](http://stackoverflow.com/questions/1995113/strangest-language-feature/2008728#2008728) made me realize that <code class="codecolorer text default">&lt;span class="text">undefined&lt;/span></code> is not a keyword in JavaScript; it actually is a **type** that also happens to have a global variable with the same name. This means that you can change the value of <code class="codecolorer text default">&lt;span class="text">undefined&lt;/span></code> (the global variable) to be something else.

```javascript
>>> typeof undefined === "undefined"
 true
 >>> undefined = 42
 42
 >>> typeof undefined === "undefined"
 false
```

And, like shown above, the correct way to check if a variable is <code class="codecolorer text default">&lt;span class="text">undefined&lt;/span></code> is by using <code class="codecolorer text default">&lt;span class="text">typeof&lt;/span></code>.

```javascript
>>> var myVariable;
 >>> typeof myVariable
 "undefined"
 >>> myVariable = 1;
 1
 >>> typeof myVariable
 "number"
```

[The second feature](http://stackoverflow.com/questions/1995113/strangest-language-feature/2010740#2010740), is something that I often used but had no idea why it actually worked: <code class="codecolorer text default">&lt;span class="text">void(0)&lt;/span></code>.

```html
<a href="javascript:void(0)">do nothing</a>
```

According to Breton's excellent explanation, <code class="codecolorer text default">&lt;span class="text">void&lt;/span></code> is actually a prefix operator. When you prefix any expression with <code class="codecolorer text default">&lt;span class="text">void&lt;/span></code>, the result evaluates to <code class="codecolorer text default">&lt;span class="text">undefined&lt;/span></code>. So, using our _correct_ method for checking <code class="codecolorer text default">&lt;span class="text">undefined&lt;/span></code> above you get this in the console:

```javascript
>>> typeof void() === "undefined"
 true
```

Awesome!