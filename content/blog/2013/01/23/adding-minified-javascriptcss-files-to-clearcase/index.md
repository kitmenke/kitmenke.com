---
author: Kit
categories:
- Uncategorized
date: 2013-01-23T15:24:29Z
guid: http://kitmenke.com/blog/?p=536
id: 536
title: Adding minified javascript/css files to ClearCase
---

I had an issue trying to add some minified JavaScript files to ClearCase.

```
Error adding '[file]' to source control.
 Created branch "dev" from "[file]" version "\main\0".
 Type manager "text_file_delta" failed create_version operation.
```

It looks like files that have over 8000 characters on a single row are unable to be added to version control using normal methods.

Thankfully, I was able to [find a solution here](http://forum.jquery.com/topic/jquery-clearcase-error-when-trying-to-add-jquery-1-2-6-min-js-to-source-control):

```
cleartool mkelem -eltype compressed_file jquery-1.2.6.min.js
 cleartool ci -nc jquery-1.2.6.min.js
```

Adding the file to clearcase as a &#8220;compressed file&#8221; means it doesn't try to do a file delta!