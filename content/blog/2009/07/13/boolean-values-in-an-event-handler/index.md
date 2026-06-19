---
aktt_notify_twitter:
- true
aktt_tweeted:
- 1
author: Kit
categories:
- SharePoint
date: 2009-07-13T14:43:05Z
guid: http://kitmenke.com/blog/?p=99
id: 99
tags:
- SharePoint 2007
title: Boolean values in an Event Handler
---

While researching a bug with editing in datasheet view and a custom list event handler, I discovered strange behavior with boolean values. Any boolean value you get from the SPItemEventDataCollection depends on where the item was originally edited: either using the regular user interface (EditForm.aspx, NewForm.aspx) or using datasheet view.

Here is the code to get a boolean item out of the list:

```csharp
private static bool GetItemColumnAsBoolean(SPItemEventDataCollection item, string columnName)
{
    bool value = false;

    if (null != item[columnName])
    {
        string booleanString = item[columnName].ToString();

        // values are different in datasheet view
        if (null != booleanString)
        {
            booleanString = booleanString.ToLower().Trim();
            switch (booleanString)
            {
                case "-1": // user changed a boolean field to true in datasheet
                case "1": // field was already true in the item after a datasheet edit
                case "true": // normal edit (not using datasheet)
                    value = true;
                    break;
            }
        }
    }

    return value;
}
```

When an item was edited using the datasheet view you will be returned integer values:

  * 0 for False
  * -1 if the user clicks the checkbox in datasheet view and saves
  * 1 if there was a boolean value anywhere in the item (and its value was already true)

Normally, you are just returned &#8220;True&#8221; or &#8220;False&#8221;.