---
aktt_notify_twitter:
- false
author: Kit
categories:
- SharePoint
date: 2009-08-18T20:55:00Z
guid: http://kitmenke.com/blog/?p=108
id: 108
tags:
- SharePoint 2007
title: Creating a SharePoint Alert with different EventTypes
---

I recently ran into a problem creating alerts in a web part for the current user. After the alert was added, no matter what I set the SPAlert's EventType to, it always defaulted to All. I finally figured out a way around this problem thanks to a helpful post by [erwin at sharepointology](http://www.sharepointology.com/development/how-to-create-alerts-programmatically/).

The workaround involves setting the eventtypeindex in the SPAlert Properties. Since I designed my web part to be configurable, an admin is able to specify the following web part properties

  * SPEventType WPPAlertEventType
  
    (Add, All, Delete, Discussion, Modify)
  * SPAlertFrequency WPPAlertFrequency
  
    (Daily, Immediate, Weekly)
  * int WPPAlertTimeHour
  
    (the hour of the day 0 &#8211; 23, only applies to Daily and Weekly alerts)
  * int WPPAlertTimeDayOfWeek
  
    (the day of the week 0 &#8211; 6, only applies to Weekly alerts)
  * bool WPPSendConfirmationEmail
  
    (whether or not to send the email saying that they were signed up for the alert)

Unfortunately, you can't just convert WPPAlertEventType and put it in &#8220;eventtypeindex&#8221; because SPEventType's enums were defined to have wacky binary values.

```csharp
// Create an Alert for this List
 SPAlert alert = currentUser.Alerts.Add();
 alert.AlertType = SPAlertType.List;
 alert.AlertTemplate = list.AlertTemplate;
 alert.AlwaysNotify = false;
 
 // DOESN'T WORK SETTING THE EVENT TYPE
 //alert.EventType = (SPEventType)WPPAlertEventType;
 
 // you must set the eventtypeindex property
 switch (WPPAlertEventType)
 {
      case SPEventType.All:
           alert.Properties["eventtypeindex"] = "0";
           break;
      case SPEventType.Add:
           alert.Properties["eventtypeindex"] = "1";
           break;
      case SPEventType.Modify:
           alert.Properties["eventtypeindex"] = "2";
           break;
      case SPEventType.Delete:
           alert.Properties["eventtypeindex"] = "3";
           break;
      case SPEventType.Discussion:
           alert.Properties["eventtypeindex"] = "4";
           break;
      default:
           break;
 }
 
 alert.AlertFrequency = WPPAlertFrequency;
 
 // get the config for daily/weekly alerts
 
 // the hours can be anything from 0 to 23
 bool alertTimeHourIsValid = (WPPAlertTimeHour &gt;=  &amp;&amp; WPPAlertTimeHour &lt;= 23);
 int alertTimeHour = alertTimeHourIsValid ? WPPAlertTimeHour : DEFAULT_WPPAlertTimeHour;
 
 // the day of the week can be anything 0 to 6 (Sunday = 0 - Saturday = 6)
 bool alertTimeDayOfWeekIsValid = (WPPAlertTimeDayOfWeek &gt;=  &amp;&amp; WPPAlertTimeDayOfWeek &lt;= 6);
 int alertTimeDayOfWeek = alertTimeDayOfWeekIsValid ? WPPAlertTimeDayOfWeek : DEFAULT_WPPAlertTimeDayOfWeek;
 
 switch (WPPAlertFrequency)
 {
      case SPAlertFrequency.Daily:
           DateTime dailyAlertTime = new DateTime(DateTime.Today.Year, DateTime.Today.Month, DateTime.Today.Day, alertTimeHour, , );
           alert.AlertTime = dailyAlertTime;
           break;
      case SPAlertFrequency.Immediate:
           break;
      case SPAlertFrequency.Weekly:
           DateTime weeklyAlertTime = new DateTime(DateTime.Today.Year, DateTime.Today.Month, DateTime.Today.Day, alertTimeHour, , ); ;
           weeklyAlertTime = weeklyAlertTime.AddDays(alertTimeDayOfWeek - (double)weeklyAlertTime.DayOfWeek);
           alert.AlertTime = weeklyAlertTime;
           break;
      default:
           break;
 }
 alert.List = list;
 alert.Title = string.Format("{0} ({1} {2})", list.Title, WPPAlertEventType, WPPAlertFrequency);
 bool sendConfirmationEmail = WPPSendConfirmationEmail;
 alert.Update(sendConfirmationEmail);
```

References:

  * [How To Create Alerts Programmatically](http://www.sharepointology.com/development/how-to-create-alerts-programmatically/)