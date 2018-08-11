---
layout: post
title:  Transparent Window in Gtk+3
date:   2017-11-02 20:27:00 +0500
categories: Programing
tags: Python Gtk+3
description: Example of how to make a fully transparent window in Gtk+3.
---

Somebody on the #gtk IRC channel asked if it was possible to make a Gtk window
transparent. The general consensus was that is was no possible, which of course
compelled me to prove otherwise. Here is the solution, which turned out not to
be very complicated, but took some time to figure out.

Not sure what this could be used for. Maybe a little tool for selecting a
region for taking a screen capture?

### Result

{% include image.html name="transparent-window.png" %}


### Code

<figure class="lineno-container">
<figcaption>transparent_window.py</figcaption>
{% highlight python linenos %}
#!/usr/bin/env python

import cairo
import gi

gi.require_version('Gtk', '3.0')
gi.require_version('Gdk', '3.0')

from gi.repository import Gtk
from gi.repository import Gdk

class TransparentWindow(Gtk.Window):
    def __init__(self):
        Gtk.Window.__init__(self)

        self.set_size_request(300, 220)

        self.connect('destroy', Gtk.main_quit)
        self.connect('draw', self.draw)

        screen = self.get_screen()
        visual = screen.get_rgba_visual()
        if visual and screen.is_composited():
            self.set_visual(visual)

        self.set_app_paintable(True)
        self.show_all()

    def draw(self, widget, context):
        context.set_source_rgba(0, 0, 0, 0)
        context.set_operator(cairo.OPERATOR_SOURCE)
        context.paint()
        context.set_operator(cairo.OPERATOR_OVER)

TransparentWindow()
Gtk.main()
{% endhighlight %}
</figure>

The example file can be downloaded from the GitHub Gist located 
[here](https://gist.github.com/KurtJacobson/374c8cb83aee4851d39981b9c7e2c22c).
