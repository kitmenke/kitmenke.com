---
author: Kit
categories:
- Uncategorized
date: 2014-11-25T14:31:56Z
guid: http://kitmenke.com/blog/?p=723
id: 723
tags:
- Powershell
- SharePoint
- SSRS
title: Powershell script to change SSRS Subscription Owner
---

We leverage Reporting Services subscriptions a lot to deliver reports via email. A problem we run into frequently is when the user who created the subscription loses permission (leaving the company, changing jobs, etc.) then all of the subscriptions they created stop working. This is because they are the **Owner** of the subscription and SSRS won't allow the subscription to run when the owner no longer has permission.

Here is a quick powershell script you can use to change the owner using the [ReportingService2010 web service endpoint](http://msdn.microsoft.com/en-us/library/reportservice2010.reportingservice2010.changesubscriptionowner.aspx).

```powershell
Write-Host '*************************************'
Write-Host 'Change Subscription Owner starting...'
Write-Host '*************************************'
$oldOwner = "domain\user1"
Write-Host "Old Owner = $oldOwner"
$newOwner = "domain\user2"
Write-Host "New Owner = $newOwner"
$itemPathOrSiteURL = "http://servername/sites/subsite"
Write-Host "Item Path or Site URL = $itemPathOrSiteURL"
$reportServerUri = "http://servername/_vti_bin/ReportServer/ReportService2010.asmx?wsdl"
Write-Host "Report Server URI = $reportServerUri"

$proxy = New-WebServiceProxy -Uri $reportServerUri -UseDefaultCredential;
$proxy.Timeout = 120000

Write-Host "Listing subscriptions for $itemPathOrSiteURL..."
$subscriptions = $proxy.ListSubscriptions($itemPathOrSiteURL)
$num = $subscriptions.length
Write-Host "Found $num subscriptions!"

$num = 0
Write-Host "Looking for subscriptions where owner = $oldOwner..."
foreach ($subscription in $subscriptions) {
    if ($subscription.Owner -eq $oldOwner) {
        $subscriptionId = $subscription.SubscriptionID
        Write-Host "   ChangeSubscriptionOwner($subscriptionId, $newOwner)"
        Try {
            $proxy.ChangeSubscriptionOwner($subscriptionId, $newOwner)
            $num += 1
        } Catch {
            Write-Host "Error changing owner: $_"
        }
    }
}
Write-Host "Successfully changed $num subscription owners!"
```

Note: This script has to run using an admin account.