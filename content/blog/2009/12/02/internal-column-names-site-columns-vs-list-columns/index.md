---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-12-02T16:02:32Z
guid: http://kitmenke.com/blog/?p=166
id: 166
title: 'Internal column names: Site Columns vs List Columns'
---

Two identical custom lists with the same columns. The only difference is that one list has columns added using Site Columns and the other list had columns added directly to the list. The same? Not so much...
  

Steps to repro this issue:

  1. Create a site column named &#8220;A,B-C/D&#8221; (single line of text or any other type)
  2. Create a list named &#8220;Special Characters (Site)&#8221;
  3. Add the Site Column &#8220;A,B-C/D&#8221; to your &#8220;Special Characters (Site)&#8221; list.
  4. Create another list named &#8220;Special Characters (List)&#8221;
  5. Create a column in &#8220;Special Characters (List)&#8221; named the same as your site column (&#8220;A,B-C/D&#8221;)

Results:

<table border="1">
  <tr>
    <td>
       
    </td>
    
    <td>
      Column Added Using Site Column
    </td>
    
    <td>
      Column Created Directly in the List
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Column Name</strong>
    </td>
    
    <td>
      A,B-C/D
    </td>
    
    <td>
      A,B-C/D
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Internal Column Name</strong>
    </td>
    
    <td>
      A_x002C_B_x002d_C_x002F_D
    </td>
    
    <td>
      A_x002c_B_x002d_C_x002f_D
    </td>
  </tr>
</table>

The internal column names are DIFFERENT.

... but only for some of the characters. Looks like the comma and the slash are different but not the dash.