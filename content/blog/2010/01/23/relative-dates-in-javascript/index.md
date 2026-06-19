---
aktt_notify_twitter:
- false
author: Kit
categories:
- JavaScript
date: 2010-01-23T22:37:41Z
guid: http://kitmenke.com/blog/?p=183
id: 183
tags:
- Example
- JavaScript
- Search
title: Relative Dates in JavaScript
lastmod: 2018-08-14T20:27:06Z
---

I was developing an input form for a search when I came across the need to allow the user to filter items using a date range. I had two input boxes where the user could enter a start and an end date. For some of the most common date ranges, I decided to create links that could be clicked to auto-populate the start/end date textboxes. For example, searching for items that were created last week or last month. Using JavaScript's Date object, I was able to put together some nice examples.

First, the setup. I needed two text boxes to hold the start/end dates and then wanted the user to click links to auto-populate them with some relative dates:

```html
<a href="javascript: void(0);" onclick="return setRelativeDate('today')">Today</a>

<label for="txtStartDate">Start Date:</label><input id="txtStartDate" type="text" />
<label for="txtEndDate">End Date:</label><input id="txtEndDate" type="text" />
```

Then to figure out how to calculate the dates using JavaScript. Here are the examples.. starting out easy with **Today**&#8216;s date. When creating a new Date object, it is automatically initialized to the current day and time:

```javascript
var today = new Date();
txtStartDate.value = getShortDateString(today);
txtEndDate.value = txtStartDate.value;
```

**Yesterday** is Today -1:

```javascript
var yesterday = new Date();
yesterday.setDate(yesterday.getDate()-1);
txtStartDate.value = getShortDateString(yesterday);
txtEndDate.value = txtStartDate.value;
```

**This Week** is a little more complicated. getDate() returns the day of the month (1-31) and getDay() returns the day of the week (0-6).

```javascript
var thisWeek = new Date();
// set the date to most recent Sunday
thisWeek.setDate(thisWeek.getDate() - thisWeek.getDay());
txtStartDate.value = getShortDateString(thisWeek);
// set the date to last week's Saturday
thisWeek.setDate(thisWeek.getDate() + 6);
txtEndDate.value = getShortDateString(thisWeek);
```

**Last Week** is pretty similar to this week. I think the hardest part for me is the function names getDate and getDay. I always was getting them confused and thought that they should have been more specific... like getWeekDay and getMonthDay or something.

```javascript
var lastWeek = new Date();
// set the date to last week's Sunday
lastWeek.setDate(lastWeek.getDate() - lastWeek.getDay() - 7);
txtStartDate.value = getShortDateString(lastWeek);
// set the date to last week's Saturday
lastWeek.setDate(lastWeek.getDate() + 6);
txtEndDate.value = getShortDateString(lastWeek);
```

The main difficulty I had with **This Month** was how to figure out the last day of the month. Luckily, if you can figure it out by selecting the first of next month and then subtracting a day.

```javascript
var lastDayOfThisMonth = new Date();
// go to the next month
var nextMonth = lastDayOfThisMonth.getMonth() + 1;
lastDayOfThisMonth.setMonth(nextMonth);
// go to the last day of the previous month (this month)
lastDayOfThisMonth.setDate();
txtEndDate.value = getShortDateString(lastDayOfThisMonth);
lastDayOfThisMonth.setDate(1);
txtStartDate.value = getShortDateString(lastDayOfThisMonth);
```

Notice how **Last Month** is actually easier because you can simply move backwards a month.

```javascript
var lastDayOfLastMonth = new Date(); // now
lastDayOfLastMonth.setDate(); // last day of last month
// set the end date first
txtEndDate.value = getShortDateString(lastDayOfLastMonth);
lastDayOfLastMonth.setDate(1);
txtStartDate.value = getShortDateString(lastDayOfLastMonth);
```

The only issue I had with **This Year** was I didn't realize that setMonth(int) took a zero based month: 0 = January ... 11 = December.

```javascript
// -1 means last year
// 11 means December and there always is a 31st of december
var thisYear = lastDayOfLastYear.getFullYear();
lastDayOfLastYear.setFullYear(thisYear - 1, 11, 31);
txtEndDate.value = getShortDateString(lastDayOfLastYear);
lastDayOfLastYear.setMonth(); // January
lastDayOfLastYear.setDate(1); // 1st
txtStartDate.value = getShortDateString(lastDayOfLastYear);
```

Finally, to figure out **Last Year** I took advantage of the setFullYear(year,month,day) function to subtract one year.

```javascript
var lastDayOfLastYear = new Date();
// -1 means last year
// 11 means December and there always is a 31st of december
var thisYear = lastDayOfLastYear.getFullYear();
lastDayOfLastYear.setFullYear(thisYear - 1, 11, 31);
txtEndDate.value = getShortDateString(lastDayOfLastYear);
lastDayOfLastYear.setMonth(); // January
lastDayOfLastYear.setDate(1); // 1st
txtStartDate.value = getShortDateString(lastDayOfLastYear);
```

See all of it together in the [complete demo](/uploads/2010/01/javascript_relativedates.html).

References:

  * [MDN Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)