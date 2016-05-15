---
layout: post
title: ImageView scale types
description: "In this post we will look into all the different scaleTypes available for the ImageView class and what they actually do."
modified: 2014-07-27
category: blog
tags: [android, tip, software development, imageview, scaletype]
share: true
---

When you build an app you have to think about thousand different things and probably you are like me and forget all the time the small details about a particular class of the framework. A common example is showing images in your app. Sometimes its just enough to add an ImageView to the layout and don't care too much about all its specific parameters, but most of the time you'll need to set up at least its ScaleType. This post does not present anything novel about this class, but I'm writing it as a reminder for myself and you about common ImageView configurations and some wanted or unwanted effects resulting from them.

### The theory - what is the ImageView's ScaleType?

![Pro Android Graphics]({{ site.url }}/images/2014-07-26-imageview-scale-types/img_1.jpg)
{: .pull-left}
I'm not going to pretend I know all theory by heart. I look it up all the time myself. Although the official documentation is really good, I have found another resource which I recommend - a book by Wallace Jackson called "Pro Android Graphics". The following definitions and descriptions are based on the information in chapter 8, dedicated to the ImageView class.

The Android ImageView has a nested class called ScaleType, which contains scaling constants, and related algorithms, for determining how to scale an ImageView object. It is most useful for determining how to scale an image relative to different screen aspect ratio, that is, keeping the aspect ratio locked so as not to distort the image at all, or, on the other end of the spectrum, allowing the image scaling operation to rescale our image without regards to aspect ratio. ScaleType is an Enum with the following possible values:

* **CENTER** - no scaling of the image, just centering it and keeping its aspect ratio. This means that if it is larger than the display, it will be cropped, if smaller it will be padded with background color.
* **CENTER_CROP** - centers the image in the display and keeps its aspect ratio. Image is scaled up or down to fit its shorter side to the display. The longer side of the image is cropped.
* **CENTER_INSIDE** - centers the image inside the display, and keeps its aspect ratio. It fits its longer side and pads the shorter side with an equal amount of background colored pixels. If the image's long side is smaller than the display this does the same thing as CENTER.
* **FIT_CENTER** - behaves like CENTER_INSIDE, except for the case when the image's longer side is smaller than the display, when it will scale up the image in order to fit its longer side to the display.
* **FIT_START** - behaves like FIT_CENTER, but does not center the image. Instead its top left corner is aligned with the display's top left corner.
* **FIT_END** - behaves like FIT_CENTER, but does not center the image. Instead its bottom right corner is aligned with the display's bottom right corner.
* **FIT_XY** - the only one that that unlocks aspect ratio and will fit the image to the size of the display, which may cause some distortion, so be careful with this one.
* **MATRIX** - if none of the other 7 works for you, you can always use this one and provide your own scaling by assigning the output of a Matrix class transformation (Rotate, Scale, Skew, etc.) using a .setImageMatrix() method call.

### Time to play - try out the different options in the companion app
I have included a demo in the blog's companion app where you can test everything you would like. You can find the app and its source code under the following links:

* [**Testground at Google Play**](https://play.google.com/store/apps/details?id=com.luboganev.testground)
* [**Testground source code at GitHub**](https://github.com/luboganev/testground)

The demo related to this post provides the following options:

* You can load into the ImageView one from 3 Hi-res image assets (one landscape, one portrait and one square) and their low resolution versions.
* The ImageView's background is grey so that you also can see where the boundaries of the view itself are.
* You can change the ScaleType of the ImageView where the images are loaded.
* You can change the width and height of the ImageView where the images are loaded.

Have fun!
