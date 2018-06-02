---
layout: post
title: Bringing slow motion to Pitivi
category: [GNOME, Pitivi]
comments: true
---

## GSoC again :)

Last year, I worked on the project ['Pitivi: Color correction interface using three chromatic wheels'](https://wiki.gnome.org/Outreach/SummerOfCode/2017/Projects/SuhasNayak_PitiviColorCorrectionInterface) as part of my Google Summer of Code. This year again, I'm working on Pitivi under the GNOME organisation. Mathieu Duponchellle and Thibault Saunier are mentoring my project this time. 

For the past couple of weeks, I've been hacking on GStreamer Editing Services (GES), Pitivi's backend, to add the 'rate' property to a clip. This is the first step towards my project ['Slow-motion Video'](https://wiki.gnome.org/Outreach/SummerOfCode/2018/Projects/SuhasNayak_PitiviSlowMotionVideo) which has two objectives:

* Add the clip speed feature to Pitivi
* Allow parts of a single clip to have variable speeds.

## A closer look

The newly introduced 'rate' property to a clip uses two GStreamer effects - videorate and pitch to modify the rate of video and audio respectively. GES already facilitates adding and maintaining effects. These effects while relying upon the existing framework to work are kept hidden from the user as internal effects.  

A [GESClip](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESClip.html) is a subclass of [GESTimelineElement](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-editing-services/html/GESTimelineElement.html), it therefore has the following relevant properties:

| GstClockTime start     | position (in time) of the object |
| GstClockTime inpoint    | position in the media from which the object should be used | 
| GstClockTime duration | duration of the object to be used | 
| GstClockTime maxduration | The maximum duration the object can have |

On changing the rate, the duration and max-duration change as a result. 

{% highlight c %}
  duration = input_duration/rate
  maxduration = (asset_duration - inpoint)/rate + inpoint
{% endhighlight %}

where, input_duration is the duration of the asset being used and asset_duration is the entire duration of the asset.

After having made the above change to GES, I made a simple UI in Pitivi to test out the rate feature. Speed property is now part of clip properties. Here's what it looks like

![Yuna Proj]({{ site.url }}/assets/yuna_proj.png)

I also enabled the 'rate' property of GESClip to work with ges-launch - a command line tool which can be used to create multimedia timeline and render/play it.

{% highlight bash %}
ges-launch-1.0 +clip ~/path/to/video.mp4 inpoint=10 
duration=20 rate=2.0
{% endhighlight %}

The above command plays the 20 seconds of the input video from the 10 second mark at a rate of 2.0, that is, for 10 seconds.

The complex part is to ensure that the entire infrastructure remains stable on this addition and this is what I am currently up to. I'm constantly pushing fixes to ensure that all existing features and operations in GES and Pitivi work with the rate property. Hopefully, by the next time I blog I can upload a clean demo video showcasing the rate feature in Pitivi :)

You can find my work on the issue [2202](https://gitlab.gnome.org/GNOME/pitivi/issues/2202). Feel free to ping me on #pitivi channel on freenode. Until next time.

