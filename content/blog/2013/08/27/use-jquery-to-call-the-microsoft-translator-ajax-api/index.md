---
author: Kit
categories:
- Uncategorized
tags:
- .NET
- JavaScript
date: 2013-08-27T14:02:33Z
guid: http://kitmenke.com/blog/?p=618
id: 618
title: Use jQuery to call the Microsoft Translator AJAX API
---

First, make sure you have [subscribed to the Microsoft Translator API on Azure Marketplace and registered your application Azure DataMarket](http://msdn.microsoft.com/en-us/library/hh454950.aspx). For testing, you can use the free subscription which lets you submit up to 2 million characters a month. Registering your app will create a <span style="text-decoration: underline;">Client ID</span> and a <span style="text-decoration: underline;">Client Secret</span> which you need to get authorization.

In order to protect your Client Secret, you'll need to write server-side code to get your access token. The access code is then passed to the Microsoft Translator API via AJAX.

On the server side, the easiest way to do this is to get authorization code when the page loads and populate a text or hidden input. You'll probably also want to include the &#8220;expires&#8221; property to keep track of how fresh the token is.

_Note: The auth token will expire every 10 minutes. For production, you'll need some way to refresh the token if the user has their page open for more than 10 minutes._

On the client side, the request to <http://api.microsofttranslator.com/V2/Ajax.svc/Translate> must use a specialy type of request named [JSONP](http://en.wikipedia.org/wiki/JSONP). A regular JSON GET request is not possible because it will violate the [same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy); in other words, your site is not hosted on microsofttranslator.com so your browser protects you by denying the requests.

In [Microsoft's example](http://msdn.microsoft.com/en-us/library/ff512406.aspx), they are getting around this by creating a `<script>` tag (the P in JSONP) and adding it to the page. I decided to instead leverage jQuery to make the JSONP request.

Here is the gist of loading an auth token server side and then allowing the user to translate some text from english to german (check out the [full code on github](https://github.com/kitmenke/MicrosoftTranslatorAJAX)):

```csharp
private static AdmAuthentication _admAuth = new AdmAuthentication(AdmConstants.CLIENT_ID, AdmConstants.CLIENT_SECRET);

protected void Page_Load(object sender, EventArgs e)
{
    txtAuthToken.Text = _admAuth.GetAccessToken().access_token;
    txtAuthExpires.Text = _admAuth.GetAccessToken().expires_in;
}
```
  
```JavaScript
$(window).load(function () {
    $('#btnAjaxTranslate').click(function (evt) {
        // get the text the user wants to translate
        var inputText = $('#txtAjaxInput').val();
        // get the current auth token (comes from server side code)
        var authToken = $('#txtAuthToken').val();
        // translate from english to german
        var data = {
            appId: 'Bearer ' + authToken,
            from: 'en',
            to: 'de',
            contentType: 'text/plain',
            text: inputText
        };
        
        // note the dataType and jsonp properties
        $.ajax({
            url: "http://api.microsofttranslator.com/V2/Ajax.svc/Translate",
            dataType: 'jsonp',
            data: data,
            jsonp: 'oncomplete'
        })
        .done(function (jqXHR, textStatus, errorThrown) {
            console.log('done', this, jqXHR, textStatus, errorThrown);
            // show the translation result to the user
            $('#txtAjaxOutput').text(jqXHR);
        })
        .fail(function (jqXHR, textStatus, errorThrown) {
            console.log('fail', this, jqXHR, textStatus, errorThrown);
        });
    });
});
```

&nbsp;