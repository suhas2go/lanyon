---
layout: post
title: A way to have Custom Effect UI - Pitivi
category: [GNOME, Pitivi]
comments: true
---

In my [previous post](https://suhas2go.github.io/gnome/2017/06/19/Porting-Old-Code/), I described 'how' I managed to port nekohayo and thiblahute's work towards providing an interface for adding custom widgets for effects in Pitivi. In this, I'll tell you 'what' it is that I have done in my first month of Google Summer of Code.

## The initial design

How Pitivi auto-generates UI for effects is interesting. For every GStreamer effect in Pitivi, a GtkGrid with the right type of widgets for its properties are packed and to manage changes to these widgets and map back the changes to the effect properties, a DynamicWidget class is created and subclassed for different type of widgets. EffectsPropertiesManager provides and caches UIs for editing effects, while GstElementSettingsWidget is a container class for configuring the effects. It is sad that such a unique infrastructure leads to a rather uniform UI. 

Porting nekohayo's branch I ended up with a light-weight plugin like architecture for having custom widgets for effects. Now, when a GstElementSettingsWidget got created it searched a particular directory to see if there is 'create_widget' entry point in 'foo_effect.py' files and saved references to this entry point. GstElementSettingsWidget's setElement method, which previously simply called add_widgets method to generate and show the GtkGrid, was changed 

* to call the 'create_widget' entry point if it existed.
* Otherwise, check if we have a custom UI availabe as a glade file.
* If all else fails, fallback to the auto-generation. 

![intial API diagram]({{ site.url }}/assets/custom_effects_initial_mm.jpg)

Both of these ways of having custom UI utilized the mapBuilder method of GstElementSettingsWidget to map the GStreamer element's properties
to corresponding widgets and wrapping the widgets themself with the standardized DynamicWidget API to control them.

Problems with this design - 
* This 'forced' us to have different files for each custom widget. 
* References to the entry points were stored as a class attribute of GstElementSettingsWidget.
* When Pitivi does have the 'plugins' feature, it should somehow, if required, be able to access this custom widget API, which this didn't allow.

## The current design

The solution we came up with was to add a 'create_widget' signal to EffectsPropertiesManager and connect a callback which would call the corresponding create_foo_widget method for a 'foo' effect. An accumulator stops emission of the signal when we receive a widget, again if all else fails then we fallback to the default handler of the signal which auto-generates the UI for the effect.

![current API diagram]({{ site.url }}/assets/custom_effects_current_mm.jpg)

This removed rigidity in the API, giving the option of creating custom widgets in single or multiple files, it is left up to the one creating the widget. We are no more storing reference to the create_widget methods for individual widgets. Plugins can connect to the 'create_widget' signal to provide enhancements.

## Another possible improvement
While making an example custom UI for the 'alpha' filter effect I noticed that within the custom widget for the effect, the widgets for individual properties can turn out to be same as the ones auto-generated. Having this additional feature of using a single custom widget for a particular property and auto-generating everything else would prove useful in such cases.


Although the [current API](https://phabricator.freedesktop.org/D1744) for custom widgets is up and running with tests, we at Pitivi want to have stable 1.0 release, as result, the current decision is that Google Summer of Code projects will not be part of this release.
Feel free to ping me on #pitivi channel on freenode :) Until next time. 
