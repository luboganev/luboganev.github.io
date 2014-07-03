---
layout: post
title: Image file extentions in posts
description: "This post describes a stupid mistake I made while writing a blogpost containing local image files in it."
modified: 2014-07-03
category: blog
tags: [tip, jekyll, github pages, blog, image, file extention]
share: true
---

Although writing posts in a Jekyll based static website is fairly simple, sometimes you might bump into an unexpected WTF situations. Typically you would build the website locally on your machine to see everything looks nice, before pushing it to Github Pages. However, you can never be 100% sure that everything will work there as well, since the server which hosts your Github page is probably a very different machine running a different OS with different configuration.

### What went wrong with my images in the post?
Well, everything looked fine when I built the website on my local machine and the images were shown in the post. However, when I pushed it to Github, the images suddenly could not be loaded. The WTFs/min increased exponentially as I tried almost everything - changing image names, putting them in different folders, etc. Then after an hour of fighting it, I finally figured out what the problem was.

>One does not simply take pictures with a camera and post them on a web server. 

I took the pictures with my camera, then downloaded them on my laptop, made some final touches in Photoshop and then posted them. However, somewhere along the way, the file extentions have changed to **".JPG"**. I have not noticed that and have referred to them in the blogpost with non-capital letters in the extention, i.e. **".jpg"**. This worked just fine on my local machine and the images were successfully loaded despite the extention was not exactly macthing. However, it did not work on Github Pages, and the images could not be loaded. So as rule of the thumb, always check the full names of your resource files including the extention.