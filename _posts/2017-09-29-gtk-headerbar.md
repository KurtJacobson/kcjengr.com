---
layout: post
title:  Custom Gedit Style Gtk+3 HeaderBar with Fullscreen Toggle
date:   2017-09-29 22:12:00 +0500
categories: Programing
tags: Python Gtk+3
description: Adding a fullscreen toggle button to the Gtk+3 HeaderBar widget.
---

**_Update:_** _Removing and re-adding the header bar as done in this code
results in the warning_  
```Gtk-WARNING **: gtk_window_set_titlebar() called on a realized window```  
_This can safely be ignored, but if it bothers you can silance the warning, or,
maybe better yet would be to create two header bars. One as the window title bar
(which will be hidden when the window is fullscreen), and another that is packed
at the top of the window, which will be shown only when the window is fulscreen.
This seems to be the way it is done in Gedit, and while it means you will have
two header bars for the window, only one is ever shown at once._

--

In addition to to the standard issue window controls, I wanted a fullscreen
toggle button in the title bar of [hazzy](https://github.com/kurtjacobson/hazzy),
so I sub-classed Gtk.HeaderBar and added one. I matched all the CSS style
classed so it should render properly with different themes.

### Results

{% include image.html name="light-theme.png" caption="Custom HeaderBar with Adwaita theme" %}

{% include image.html name="dark-theme.png" caption="Custom HeaderBar with dark theme" %}


### Code

<figure class="lineno-container">
<figcaption>header_bar.py</figcaption>
{% highlight python linenos %}
#!/usr/bin/env python

#  Copyright (c) 2017 Kurt Jacobson
#  License: https://kcj.mit-license.org/@2017

import os
import gi

gi.require_version('Gtk', '3.0')
gi.require_version('Gdk', '3.0')

from gi.repository import Gtk
from gi.repository import Gdk

# GtkHeaderBar source, useful for finding style class names etc.
#    https://git.gnome.org/browse/gtk+/tree/gtk/gtkheaderbar.c?h=3.22.19#n394

ICONSIZE = Gtk.IconSize.MENU

class HeaderBar(Gtk.HeaderBar):
    def __init__(self, window, *args, **kwargs):
        Gtk.HeaderBar.__init__(self, *args, **kwargs)

        self.window = window
        self.window_box = window.get_child()

        self.window_size = None
        self.window_position = None

        # Track window state
        self.window_fullscreen = False
        self.window_maximized = False
        self.window.connect('window-state-event', self.on_window_state_event)
        self.window.connect('map-event', self.on_map)

        GetIcon = Gtk.Image.new_from_icon_name

        # Close icon
        self.close_icon = GetIcon('window-close-symbolic', ICONSIZE)
        self.close_icon.show()

        # Maximize icon
        self.maximize_icon = GetIcon('window-maximize-symbolic', ICONSIZE)
        self.maximize_icon.show()

        # Restore (unaximize) icon
        self.restore_icon = GetIcon('window-restore-symbolic', ICONSIZE)
        self.restore_icon.show()

        # Minimize icon
        self.minimize_icon = GetIcon('window-minimize-symbolic', ICONSIZE)
        self.minimize_icon.show()

        # Fullscreen icon
        self.fullscreen_icon = GetIcon('view-fullscreen-symbolic', ICONSIZE)
        self.fullscreen_icon.show()

        # Unfullscreen icon
        self.unfullscreen_icon = GetIcon('view-restore-symbolic', ICONSIZE)
        self.unfullscreen_icon.show()

        # Control button box, set style class to match that the default box
        self.button_box = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL)
        style = self.button_box.get_style_context()
        style.add_class('right')

        # Add to the right side of the titlebar
        self.pack_end(self.button_box)

        # Add the separator
        self.separator = Gtk.Separator(orientation=Gtk.Orientation.VERTICAL)
        self.button_box.pack_start(self.separator, False, False, 3)
        self.separator.get_style_context().add_class('titlebutton')

        # Fullscreen button
        self.fullscreen_btn = Gtk.Button()
        self.fullscreen_btn.add(self.fullscreen_icon)

        # Add same style classes as used by default controls
        self.fullscreen_btn.get_style_context().add_class('titlebutton')
        self.fullscreen_btn.get_style_context().add_class('fullscreen')
        self.button_box.pack_start(self.fullscreen_btn, False, False, 3)
        self.fullscreen_btn.connect('clicked', self.on_fullscreen_clicked)


        # Minimize button
        self.minimize_btn = Gtk.Button()
        self.minimize_btn.add(self.minimize_icon)

        # Add same style classes as used by default controls
        self.minimize_btn.get_style_context().add_class('titlebutton')
        self.minimize_btn.get_style_context().add_class('minimize')
        self.button_box.pack_start(self.minimize_btn, False, False, 3)
        self.minimize_btn.connect('clicked', self.on_minimize_clicked)


        # Maximize button
        self.maximize_btn = Gtk.Button()
        self.maximize_btn.add(self.maximize_icon)

        # Add same style classes as used by default controls
        self.maximize_btn.get_style_context().add_class('titlebutton')
        self.maximize_btn.get_style_context().add_class('maximize')
        self.button_box.pack_start(self.maximize_btn, False, False, 3)
        self.maximize_btn.connect('clicked', self.on_maximize_clicked)


        # Close button
        self.close_btn = Gtk.Button()
        self.close_btn.add(self.close_icon)

        # Add same style classes as used by default controls
        self.close_btn.get_style_context().add_class('titlebutton')
        self.close_btn.get_style_context().add_class('close')
        self.button_box.pack_start(self.close_btn, False, False, 3)
        self.close_btn.connect('clicked', self.on_close_clicked)

    def on_map(self, widget, event):

        # Prevent error on start up
        if not self.window_size or not self.window_position:
            return

        # Restore size from before fullscreen
        w, h = self.window_size
        self.window.resize(w, h)

        # Restore position
        x, y = self.window_position
        self.window.move(x, y)

    def on_close_clicked(self, widget):
        self.window.close()

    def on_maximize_clicked(self, widget):
        if self.window_maximized:
            self.window.unmaximize()
        else:
            self.window.maximize()

    def on_minimize_clicked(self, widget):
        self.window.iconify()

    def on_fullscreen_clicked(self, widget):
        if self.window_fullscreen:
            self.window.unfullscreen()
        else:
            # Get the current size and post so can restore latter
            self.window_size = self.window.get_size()
            self.window_position = self.window.get_position()
            self.window.fullscreen()

    def on_maximized_state_changed(self, maximized):
        # Remove whatever icon is present
        image = self.maximize_btn.get_child()
        self.maximize_btn.remove(image)

        if maximized:
            # Change to the restore icon
            self.maximize_btn.add(self.restore_icon)
        else:
            # Change back to the maximize icon
            self.maximize_btn.add(self.maximize_icon)

    def on_fullscreen_state_changed(self, fullscreen):
        # Remove whatever icon is present
        image = self.fullscreen_btn.get_child()
        self.fullscreen_btn.remove(image)

        if fullscreen:
            # Remove the headerbar from the titlebar and add to top of window
            self.window.remove(self)
            self.window_box.pack_start(self, False, False, 0)

            # Will end up ad the bottom, so move to top
            self.window_box.reorder_child(self, 0)

            # Change to un-fullscreen icon
            self.fullscreen_btn.add(self.unfullscreen_icon)

            # Maximizing a fullscreen window does nothing, so hide the button
            self.maximize_btn.hide()

        else:

            # Remove headerbar from box and put back in titlebar
            self.window_box.remove(self)

            # Calling `set_titlebar` on a realized window causes this (harmless) warning
            #    Gtk-WARNING **: gtk_window_set_titlebar() called on a realized window
            self.window.set_titlebar(self)

            # Change the icon back
            self.fullscreen_btn.add(self.fullscreen_icon)

            # Re-show the maximize button
            self.maximize_btn.show()

    def on_window_state_event(self, widget, event):
        # Listen to state event and track window state
        if self.window_fullscreen != bool(event.new_window_state & Gdk.WindowState.FULLSCREEN):
            self.window_fullscreen = bool(event.new_window_state & Gdk.WindowState.FULLSCREEN)
            self.on_fullscreen_state_changed(self.window_fullscreen)
        if self.window_maximized != bool(event.new_window_state & Gdk.WindowState.MAXIMIZED):
            self.window_maximized = bool(event.new_window_state & Gdk.WindowState.MAXIMIZED)
            self.on_maximized_state_changed(self.window_maximized)


def main():
    window = Gtk.Window()

    box = Gtk.Box(orientation=Gtk.Orientation.VERTICAL)
    window.add(box)

    title = "Custom HeaderBar"
    subtitle = "with fullscreen toggle button"
    header_bar = HeaderBar(window, title=title, subtitle=subtitle)
    window.set_titlebar(header_bar)

    window.set_default_size(600, 300)
    window.connect('destroy', Gtk.main_quit)

    window.show_all()
    Gtk.main()

if __name__ == "__main__":
main()
{% endhighlight %}
</figure>

The example file can be downloaded from the GitHub Gist located
[here](https://gist.github.com/KurtJacobson/6b045b6fc38907a2f18c38f6de2929e3).
