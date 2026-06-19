---
author: Kit
categories:
- Uncategorized
tags:
- SSRS
date: 2012-12-12T17:26:29Z
guid: http://kitmenke.com/blog/?p=529
id: 529
title: Top 3 Features missing from SSRS 2008 R2
---

A list of gripes about features I feel are missing from Microsoft SQL Server Reporting Services 2008 R2.

# 1. The ability to throttle an individual report

Report Builder 3.0 makes it easy to create reports which is great and encourages &#8220;self-service&#8221; reporting. Business users creating reports hooray!

Unfortunately, there is no way to prevent damage to a production environment caused by a poor-performing report. If someone creates a report that consumes 2 GB of memory on the SSRS server  there is no way to prevent that report from running short of disabling the datasource or removing the report entirely.

There needs to be a way to set a memory or time limit in &#8220;Manage Processing Options&#8221;.

# 2. Error handling for failed subscriptions

Subscriptions are such a great idea and users love them. However, if a subscription fails.. what happens?

Answer: nothing. Essentially, what you can do is click &#8220;Manage Subscriptions&#8221; to check the &#8220;Last Results&#8221; field. This assumes you know which report has subscriptions.

There needs to be some way to view subscriptions in a site collection, detect failed subscriptions, and provide the ability to (automatically) re-run failed subscriptions.

<span style="text-decoration: underline;">Bonus:</span> if a user creates a subscription and then their account is deactivated (example: leaving the company), their subscriptions will fail. Ouch!

# 3. Logging (AKA the ability to troubleshoot)

Troubleshooting an issue? Good luck using the logs to figure it out. Which logs do you want to look at? SharePoint logs? IIS logs? SSRS logs? Event viewer? Database logs?

Assuming you're able to gather all of that information up.. it is impossible to correlate a specific run together. I think in the newer version of SSRS there is a correlation ID.. so maybe someone can confirm.

Also, there needs to be a detailed log for Subscriptions. A &#8220;Last Results&#8221; field that gets overwritten on subsequent runs is next to useless.