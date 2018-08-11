---
layout: post
title:  Dragable widgets in Gtk+3
date:   2017-10-16 19:00:00 +0500
categories: Programing
tags: Python Gtk+3
description: Example showing how to make it possible to drag/reposition GTK+ widgets with the mouse.
---

### Introduction

A simple example showing how to make it possible to drag and reposition GTK+
widgets with the mouse. Code very similar to this is used the the Drag-n-Drop
screen layout editor in [hazzy](https://github.com/kurtjacobson/hazzy).

### Synopsis

On _button_press_event_ (called once at drag begin) get the initial location of the pointer
and calculate the maximum change in position of the widget such that is will
not be moved outside of the border of its parent window.

On _motion-notify_event_ (called during drag motion) calculate the change in position of the pointer and
determine the position to move the widget to, taking into account that the widget
must not exceed the bounds of its parent.

<div class="alert alert-info" role="alert">
    <strong>Note:</strong><br>
    To avoid having to take into account window border offsets, we use the change
    in position of the pointer rather than the absolute position for determining
    how much to move the widget. This is extremely important for making the code
    work reliably with different window managers and window types.
</div>

### Result

{% include image.html name="move_btn.gif" %}

### Example Code

<figure class="lineno-container">
<figcaption>move_button.py</figcaption>
{% highlight python linenos %}
#!/usr/bin/env python

import gi

gi.require_version('Gtk', '3.0')
from gi.repository import Gtk

class MoveButton(Gtk.Fixed):

    def __init__(self):
        Gtk.Fixed.__init__(self)

        # Initial event position
        self.initial_x = 0
        self.initial_y = 0

        # Initial button position
        self.initial_pos_x = 0
        self.initial_pos_y = 0


    # Called on "drag start"
    def button_press_event(self, widget, event):

        # Get the parent window size
        pw = self.get_allocation().width
        ph = self.get_allocation().height

        # Get the button location and size
        x = self.child_get_property(widget, 'x')
        y = self.child_get_property(widget, 'y')
        w = widget.get_allocation().width
        h = widget.get_allocation().height

        # Calculate the maximum change in position such that the
        # button does not move outside the bounds of its parent
        self.dx_max = pw - (x + w)
        self.dy_max = ph - (y + h)

        # Save the initial values
        self.initial_x = event.x_root
        self.initial_y = event.y_root
        self.initial_pos_x = x
        self.initial_pos_y = y

    # Called on "drag motion"
    def motion_notify_event(self, widget, event):

        # Calculate the change in position of the pointer
        dx = event.x_root - self.initial_x
        dy = event.y_root - self.initial_y

        # Determine the new position such that the button
        # does not move outside the bounds of its parent
        x = self.initial_pos_x + max(min(dx, self.dx_max), -self.initial_pos_x)
        y = self.initial_pos_y + max(min(dy, self.dy_max), -self.initial_pos_y)

        # Actualy move the button
        self.child_set_property(widget, 'x', x)
        self.child_set_property(widget, 'y', y)


def make_button(text):

    # Helper function to make the buttons
    b = Gtk.Button.new_with_label(text)
    b.connect("button_press_event", move_btn.button_press_event)
    b.connect("motion_notify_event", move_btn.motion_notify_event)
    b.show()
    return b

# New top level window
window = Gtk.Window()
window.connect("destroy", Gtk.main_quit)

# Initialize the MoveButton container
move_btn = MoveButton()
window.add(move_btn)

# Add some buttons
move_btn.put(make_button("A button"), 50, 50)
move_btn.put(make_button("Another button"), 250, 100)

window.show_all()

# Start the main loop
Gtk.main()
{% endhighlight %}
</figure>
