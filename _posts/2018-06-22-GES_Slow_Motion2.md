---
layout: post
title: Behind the GESSourceClip rate
category: [GNOME, Pitivi]
comments: true
---

## Initial Approach

GES has an effects infrastructure for adding and managing GStreamer elements. Since the rate property uses [videorate](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gst-plugins-base-plugins/html/gst-plugins-base-plugins-videorate.html) and [pitch](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gst-plugins-bad-plugins/html/gst-plugins-bad-plugins-pitch.html) elements to work, the idea was to use this existing infrastructure but to the hide the effects from the user as they are required only internally.

### Step 1: Use effects

All media elements are truly set on a clip only when it is added to a layer, this involves the construction of [GESAudioSource](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESAudioSource.html) and/or [GESVideoSource](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESVideoSource.html) of the clip. An effect can be added only after these sources have been added to the clip. As a result, the rate property was configured to add pitch and/or videorate as effects only after the sources were added. 

### Step 2: Hide the effects

To hide the effects from the user - accommodate the hidden effects in GESClip by mimicing the behaviour of a normal effect and maintain a reference to the 'hidden' rate changing effects to not display them as top effects. 

A [GESClip](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESClip.html) is a subclass of [GESContainer](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESContainer.html) - which gives it the ability to hold the source track elements and the effects. Many operations in GES call the [GES_CONTAINER_CHILDREN (clip)](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESContainer.html#GES-CONTAINER-CHILDREN:CAPS) method when required to retrieve and make changes to the children of a clip - effects/sources contained in the clip. Since the rate changing effects were not truly hidden, they are contained in the clip and hence showed up as the container's children.

### Step 3: Start from scratch

Although the above approach of maintaining hidden effects works to hide them from the user, to hide and make GES sometimes ignore and sometimes utilise the effect API was not only difficult but required making a lot of ugly changes to many parts of GES. It became obvious that a rework of the way things worked was required when running tests on the branch resulted in most of the existing tests failing.  

## Current Implementation

Mathieu suggested that instead of using the effect API, we should dig a little deeper and add the videorate and pitch elements to the sources of the clip ourself. This genius and simple solution eliminated many of the problems faced in the initial approach.

pitch is simply added to the audiosrcbin- a [GstBin](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer/html/GstBin.html) of GESAudioSource, while videorate was already in place in a GESVideoSource to adjust frames, as a result, rate is now added as a child property of the audio and video source in GES. The parent [GESSource](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESSource.html) handles creating, linking and managing the audiosrcbin and videosrcbin. Rate property added to [GESSourceClip](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESSourceClip.html) - base class for sources of a [GESLayer](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESLayer.html), changes the rate child property of its sources to function.

The pitch element being from the gst-plugins-bad didn't behave so well and required some fixes for [bug 796603](https://bugzilla.gnome.org/show_bug.cgi?id=796603) by Matheiu and a followup fix for [bug 796613](https://bugzilla.gnome.org/show_bug.cgi?id=796613) by myself after which all existing tests of GES passed! (hurray)

While pushing new tests, I realised that since we no longer rely on the effect API, the rate property had to be serialised and deserialised by adding to the xges xml formatter. 

Same functionality with ges-launch holds, simply adding rate to command changes speed of a clip.

{% highlight bash %}
ges-launch-1.0 +clip ~/path/to/video.mp4 inpoint=10 duration=20 rate=2.0
{% endhighlight %}

The above command plays the 20 seconds of the input video from the 10 second mark at a rate of 2.0, that is, for 10 seconds.

The project can saved and loaded back-in using the following commands, 

{% highlight bash %}
ges-launch-1.0 +clip ~/path/to/video.mp4 inpoint=10 duration=20 rate=2.0 --save project.xges

ges-launch-1.0 --load project.xges
{% endhighlight %}

The primary focus of work has been on the getting the GES implementation right. I'm yet to try out a few suggestions I got from GNOME design on the Pitivi UI side of things.

This has been the story so far. You can find my work [here](https://gitlab.gnome.org/suhas2go/gst-editing-services/merge_requests/1) and follow issue [2202](https://gitlab.gnome.org/GNOME/pitivi/issues/2202) for updates. Until next time.




