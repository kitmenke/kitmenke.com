---
aktt_notify_twitter:
- true
aktt_tweeted:
- 1
author: Kit
categories:
- SharePoint
date: 2009-09-05T01:10:43Z
guid: http://kitmenke.com/blog/?p=138
id: 138
tags:
- SharePoint 2007
title: SharePoint's SPGridView, filtering, and apostrophes
---

**May 20, 2010 Update**
  
If you are looking for a great example on how to create an SPGridView, check out Erik Burger's blog:

  1. [Building A SPGridView Control – Part 1: Introducing the SPGridView](http://reversealchemy.nl/building-a-spgridview-control-part-1-introducing-the-spgridview/)
  2. [Building A SPGridView Control – Part 2: Filtering](http://reversealchemy.nl/building-a-spgridview-control-part-2-filtering/)

This post is really meant to supplement part 2 of Erik's guide and address values in the grid that include apostrophes when using filtering.

* * *

The SPGridView is one of the most useful SharePoint controls. You can use it to do sorting, grouping, and filtering just like the out of the box List View Web Part does for regular lists and libraries. This makes it relatively easy to create a custom grid with your own data. Unfortunately, filtering with the SPGridView is a little quirky, especially if your data could potentially contain apostrophes.

Essentially, all you need to do is set a few quick properties to enable filtering on the SPGridView. (

[see the great tutorial on Reverse Alchemy for more information](http://reversealchemy.nl/building-a-spgridview-control-part-2-filtering/))

```csharp
// Filtering
grid.AllowFiltering = true;
grid.FilterDataFields = ",Name,Region,Total Sales";
grid.FilteredDataSourcePropertyName = "FilterExpression";
grid.FilteredDataSourcePropertyFormat = "{1} = '{0}'";
```

[<img class="alignnone size-full wp-image-144" title="2009-09-03 14 04 35" alt="2009-09-03 14 04 35" src="/uploads/2009/09/2009-09-03-14-04-35.png" width="502" height="170" srcset="/uploads/2009/09/2009-09-03-14-04-35.png 502w, /uploads/2009/09/2009-09-03-14-04-35-300x101.png 300w" sizes="(max-width: 502px) 100vw, 502px" />](/uploads/2009/09/2009-09-03-14-04-35.png)

However, what happens if you have an apostrophe for the value in one of your filters (for example filtering on the name of O'Reilly). Eventually, you will end up with a filter of:

```
Name = 'O'Reilly'
```

If this is the case, when you attempt to filter using that value you will get the error:

```
Syntax error: Missing operand after 'Reilly' operator
```

([See the question asked by Moo on StackOverflow](http://stackoverflow.com/questions/1351578/spgridview-data-and-correct-method-of-ensuring-data-is-safe))

So how do you prevent the apostrophes from messing up your filter?

```csharp
protected override void OnPreRender(EventArgs e)
{
  if (!string.IsNullOrEmpty(_gridDS.FilterExpression))
  {
    _gridDS.FilterExpression = string.Format(
    _grid.FilteredDataSourcePropertyFormat,
    _grid.FilterFieldValue.Replace("'", "''"),
    _grid.FilterFieldName);
  }
  base.OnPreRender(e);
}
```

[<img class="alignnone size-full wp-image-145" title="2009-09-03 14 07 58" alt="2009-09-03 14 07 58" src="/uploads/2009/09/2009-09-03-14-07-58.png" width="372" height="86" srcset="/uploads/2009/09/2009-09-03-14-07-58.png 372w, /uploads/2009/09/2009-09-03-14-07-58-300x69.png 300w" sizes="(max-width: 372px) 100vw, 372px" />](/uploads/2009/09/2009-09-03-14-07-58.png)

Where \_gridDS is an ObjectDataSource (you must use an ObjectDataSource as opposed to setting the DataSource SPGridView property) and \_grid is your SPGridView.

The solution is relatively simple... but what is happening behind the scenes? Actually, pretty much the same thing but without the replace. The SPGridView looks up your ObjectDataSource using the _DataSourceID_ property. Once it has your ObjectDataSource, it then sets the property specified in FilteredDataSourcePropertyName (FilterExpression) to be the filter. Once you understand that behind the scenes, all it really is doing is setting the FilterExpression on your datasource, it is pretty easy to override the incorrect filter with a correct one.