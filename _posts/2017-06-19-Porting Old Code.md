---
layout: post
title: Not rebasing an old feature branch
category: GNOME
comments: true
---

Ultimately, my aim was to provide an interface, at the code level, for developers (rather designers) to improve the GUI of video effects. GStreamer effects are very beautifully handled in Pitivi, the main focus was to use this existing underlying infrastructure on top of which a way of easily adding custom UI for effects had to be setup. 

One of the ways of stopping 'duplication of effort' in Open Source projects is to document everything, even failed/blocked attempts. Thanks to nekohayo (previous maintainer at Pitivi) opening task [T3263](https://phabricator.freedesktop.org/T3263), his work from 2013 towards providing such an interface is now up and running again.

![](http://lgam.wdfiles.com/local--files/duplication-of-effort/tumblr_lkbipc0gMH1qaxpypo1_1280.jpg)

Unless you want to preserve commits, rebasing a very old feature branch, that is not yours, is pointless. I did not want to waste time resolving merge conflicts in then unfamiliar code. Following a bottom-up approach, I started working on top of the current Pitivi master integrating the old code into it, step-by-step, one function at a time. Compiling and understanding the errors and then fixing them. I found this approach to be rather systematic and I think it is much faster since you start porting the code as you read it. 

After I had completed porting, it was a first time for me hitting a [git-core bug](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=451880) regarding support for multiple authors on a single commit. I simply settled with the temporary solution of using 

    Co-authored-by: Some One <some.one@example.foo>

in my commit messages.

Finally, at the end of all this I was able to get an example interface for the alpha filter built using glade to work via the above mechanism. 

![Glade UI File]({{ site.url }}/assets/port1.png)

![Working UI]({{ site. url }}/assets/port2.png)

The exact API will, most likely, undergo change. I will describe it in detail in my next post in a couple of weeks. You can checkout my work so far [D1744](https://phabricator.freedesktop.org/D1744) and [D1745](https://phabricator.freedesktop.org/D1745), feel free to ping me on #pitivi on freenode :)
