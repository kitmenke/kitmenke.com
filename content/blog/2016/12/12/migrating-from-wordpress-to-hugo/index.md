---
author: Kit
date: 2016-12-12T19:32:30-06:00
lastmod: 2018-01-17T20:22:17-06:00
categories: ["Hugo"]
tags: ["Hugo"]
title: Migrating from Wordpress to Hugo
description: Migrating from Wordpress to Hugo and the workflow for using it on Windows

---
I recently decided to migrate my blog from Wordpress to Hugo. I don't blog as often these days so trying to keep up with new wordpress versions, securing my wordpress installation against hack attempts, and fighting comment spam was becoming overwhelming. 

Here is an old screenshot from the Akismet dashboard a few years ago which shows over 215k spam comments: ![Graph of spam caught with Akistmet](/uploads/2017/01/akismet.png)

Fighting the spam is impossible to do manually so you pretty much have to pay for an [Akismet](https://akismet.com/) subscription (which works amazing by the way and definitely worth the money if you have a wordpress blog).

Also, I always felt like the editor was built for photography/writing blogs first and I had to fight to create a technical blog on wordpress. Formatting a post which contained a lot of code blocks with syntax highlighting was a chore and I eventually switched to using GitHub gist embeds entirely so that I didn't have to fight with encoding and indentation issues. It was time for a change.

# A new Workflow with Hugo

My new workflow uses the following tools (same as development!):

* Git for version control
* [Hugo](http://gohugo.io/) for website generation
* Visual Studio Code for editing (with the Insert Date String and Markdown All in One extensions)
* aws cli for deployment

To create a new post, it goes something like this:

```
hugo new post/2017-01-01-some-new-post.md
hugo server --watch
```

Then I open the new post in VS Code and edit it in markdown. The hugo server is available at `http://localhost:1313/` to see the live preview. Once I'm finished with the new post, I deploy the files to my S3 bucket:

```
aws s3 sync .\public\ s3://kitmenke.com/ --delete
```

There are a few things I miss from wordpress:

 - Creating posts is more annoying (generate slug from title, chooser for tags and categories, automatic create/modify/publish date)
 - Inserting images into posts is more work
 - More tweaking to do with themes to achieve the same functionality that are available as widgets in wordpress out of the box

## Alternative using rsync via the Bash shell on Windows 10

I used this for a while when I had a normal web host. Since moving to AWS, I don't need rsync so I don't use this but it worked well. The downside is having to maintain two versions of hugo.

How to install the Linux bash shell on windows 10:
http://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/

Don't modify the files stored in your linux filesystem, store them on windows like normal and access via /mnt/c
https://blogs.msdn.microsoft.com/commandline/2016/11/17/do-not-change-linux-files-using-windows-apps-and-tools/

Grab the latest from https://github.com/spf13/hugo/releases

Setup your SSH keys:
http://superuser.com/questions/215504/permissions-on-private-key-in-ssh-folder

## Updating Hugo

Windows

Use [chocolatey](chocolatey.org) and install the hugo package. Then updating is a breeze:
```
choco upgrade hugo
hugo version
```

Linux

Go to https://github.com/gohugoio/hugo/releases to find the latest release
```
wget https://github.com/spf13/hugo/releases/download/v0.18.1/hugo_0.18.1-64bit.deb
sudo dpkg -i hugo_0.18.1-64bit.deb
hugo version
```