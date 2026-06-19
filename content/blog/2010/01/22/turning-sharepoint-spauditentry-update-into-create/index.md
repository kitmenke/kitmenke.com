---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2010-01-22T16:42:47Z
guid: http://kitmenke.com/blog/?p=178
id: 178
tags:
- SharePoint 2007
title: Turning SharePoint SPAuditEntry Update into Create
---

If you've done any work with SharePoint and SPAuditEntry, you may have noticed the distinct lack of Create event. Instead, two Update events are created which means some extra processing is required.

For example, after uploading &#8220;NewTxtFile.txt&#8221; into a document library inside one of my sites, the following Audit Entries showed up.

```xml
>
     >9a70a244-1baa-4229-84ac-3f06bccecc7f>
     >f332a82b-17d1-4c31-9726-5259769f2695>
     >List>
     >2>
     >sites/MySiteCollection/MySubsite/Documents>
     >Url>
     >1/22/2010 9:17:06 PM>
     >Update>
     >SharePoint>
     >NewTxtFile.txt>
 >
 
 >
     >9a70a244-1baa-4229-84ac-3f06bccecc7f>
     >19ce9254-93bc-41ff-9da1-ba5e994bf1c1>
     >Document>
     >2>
     >sites/MySiteCollection/MySubsite/Documents/NewTxtFile.txt>
     >Url>
     >1/22/2010 9:17:06 PM>
     >Update>
     >SharePoint>
 >
```

Notice the &#8220;Occurred&#8221; DateTime properties are the same as well the Event. Also, one is an Update for the List and one is an Update for the Document. Using this information, you can then simplify the duplicate Update events into one Create event!

**Update 08/18/2010:**

Looks like Thomas found an easier method for determing Create events! I did some more research after seeing his comment and found the following:

  1. Creates: 
      * Document Audit Entry: EventData is null
      * List Audit Entry: EventData contains document filename
  2. Edits: 
      * Document Audit Entry: EventData contains version information

Here are the two events generated during a Create:

```xml
>
     >9a70a244-1baa-4229-84ac-3f06bccecc7f>
     >ca98cdba-c44f-4318-bb44-a46bd674d80f>
     >Document>
     >19>
     >sites/MySiteCollection/MySubsite/Documents/Test Document.doc>
     >Url>
     >8/18/2010 2:21:43 PM>
     >Update>
     >SharePoint>
 >
 
 >
     >9a70a244-1baa-4229-84ac-3f06bccecc7f>
     >28efdcba-4639-445f-80b6-31964a9f95f1>
     >List>
     >19>
     >sites/MySiteCollection/MySubsite/Documents>
     >Url>
     >8/18/2010 2:21:43 PM>
     >Update>
     >SharePoint>
     >Test Document.doc>
>
```

Here is the event that was generated after an Edit:

```xml
>
     >9a70a244-1baa-4229-84ac-3f06bccecc7f>
     >ca98cdba-c44f-4318-bb44-a46bd674d80f>
     >Document>
     >19>
     >sites/MySiteCollection/MySubsite/Documents/Test Document.doc>
     >Url>
     >8/18/2010 2:25:47 PM>
     >Update>
     >SharePoint>
     >
        >
            >1>
            >>
        >
    >
>
```

Thanks Thomas!