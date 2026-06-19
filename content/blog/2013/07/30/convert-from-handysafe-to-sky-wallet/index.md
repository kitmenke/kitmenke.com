---
author: Kit
categories:
- Uncategorized
tags:
- Python
date: 2013-07-30T23:44:32Z
guid: http://kitmenke.com/blog/?p=606
id: 606
title: Convert from HandySafe to Sky Wallet
---

I wrote a small python script to help convert Handy Safe to Sky Wallet. I definitely learned a lot about python and even though I wasn't very familiar with the available API I was able to come up with a pretty nifty script.

First you should export your data from Handy Safe using File -> Export. The file will be saved as an XML file, for example &#8220;test1.xml.&#8221;

Then, you can run <code class="text">python HandySafeXmlToSkyWalletCsv.py</code> to convert that XML file into CSV file. The generated file is [formatted for Sky Wallet](http://skywallet.net/2011/05/13/importing-data-with-desktop-companion/) and ready to be imported there. Be sure to tweak any of the variables inside the script if you want to change your filenames.

Make sure to [download the script below from Github](https://gist.github.com/kitmenke/6119212) otherwise your indentation will be messed up.

<script src="https://gist.github.com/kitmenke/6119212.js"></script>

If this was helpful or if you have any feedback on the script.. let me know!