---
author: Kit
categories:
- JavaScript
date: 2013-11-05T17:33:52Z
guid: http://kitmenke.com/blog/?p=661
id: 661
tags:
- JavaScript
- knockout
title: 'Knockout Validation: Multiple email addresses'
---

A simple rule for validating an observable which could contain multiple email addresses.

Assuming you have an observable like `self.emailTO = ko.observable('');`, you would just extend it to have `self.emailTO.extend({ multiemail: true });`. Use it with the required validator if you want at least one email address.

```javascript
ko.validation.rules['multiemail'] = {
    validator: function (val, validate) {
        if (!validate) { return true; }

        var isValid = true;
        if (!ko.validation.utils.isEmptyVal(val)) {
            // use the required: true property if you don't want to accept empty values
            var values = val.split(';');
            $(values).each(function (index) {
                isValid = ko.validation.rules['email'].validator($.trim(this), validate);
                return isValid; // short circuit each loop if invalid
            });
        }
        return isValid;
    },
    message: 'Please enter valid email addresses (separate multiple email addresses using a semicolon).'
};
```

Requires [Knockout.js](http://knockoutjs.com/), [Knockout Validation](https://github.com/Knockout-Contrib/Knockout-Validation), and jQuery.