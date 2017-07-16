---
layout: post
title: Two Realizations...
category: [GNOME, Pitivi]
comments: true
---

## How Custom Effect Widget API evolved

Simply using the initial API, for allowing custom widgets instead of the default UI for effects, has brought in a lot of changes to it. Being new to such a task of designing the API, it was good that I had to make an [example custom UI](https://phabricator.freedesktop.org/D1745) for the alpha effect. I realized that the default UI for effects does a good job (I mean, you can't do anything more fancy than the default sometimes), except for some widgets of effect properties where something more modern can provide a better user interface for editing. This called for an improvement to the existing custom widget mechanism.

Now, custom widgets for only certain properties can be added while falling back to the auto-generated UI for the rest. To maintain consistency, one has to connect a callback to the 'create_property_widget' signal and return a DynamicWidget wrapper around the custom property widget. Although, I would have liked to do away with this wrapper. It seems like the best approach as it would allow the designer to truly customize the widget.

[D1777](https://phabricator.freedesktop.org/D1777) holds these new changes.   
 	
## Baby steps in making custom widgets

For the past few days, I've been learning gtk in c and cairo. Porting the [classic EggClock](https://thegnomejournal.wordpress.com/2005/12/02/writing-a-widget-using-cairo-and-gtk2-8/) to gtk3 and exploring the basics. My next immediate task is to make a color wheel widget. Gtk3 already had such a widget, [GtkHSV](https://developer.gnome.org/gtk3/stable/GtkHSV.html), which is now deprecated (infact, removed from the master!). Having talked on the IRC, it seems Gtk had pulled the widget from GIMP but since they found that it was not being used by other applications, they decided to deprecate it. GIMP has taken back their widget. Although, our widget would heavily borrow code, it seems building a separate Pitivi Library of custom widgets would be the best thing to do.

Subclassing a GtkDrawingArea and drawing a circle, I was trying to run a skeleton-like custom widget code, all I could see was an empty window. Took me to some to 'realize' my mistake (I had left the overrided realize callback empty). I was so happy when I managed to draw this circle in cairo :P

![baby custom cairo]({{ site.url }}/assets/baby_custom_cairo.png)

Until next time :)

