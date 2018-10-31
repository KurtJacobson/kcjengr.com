---
layout: post
title:  Corvair Intake Cover
date:   2018-10-24 10:05:35 +0500
categories: Automotive
tags: 3D-Printing CAD/CAM
description: 3D printed intake manifold cover for 110Hp Corvair engines.

gallery:
  - image: in_place_1.jpg
  - image: in_place_2.jpg
  - image: in_use.jpg
  - image: sp_screenshot_1.png
  - image: sp_screenshot_2.png
---

In preparation for pressure washing the engine from my '64 Corvair Monza,
I needed to plug up the intake manifold where the carburetors mount. The
tried and true method is to use a paper towel "burrito", but it had been
a while since I had run the printer, so I decided to print some little
covers.

I decided to model the part in F360, since I am not as comfortable with it
as other CAD packages, so I booted the Windows 10 machine.

I had not booted windows for over 4 months, and since then my graphics card had
failed and been replaced. While the new card Just Worked with Debian, for some
reason windows did not like it so the DPI scaling was WAY off. I spent about 45
min trying to find a driver for fix the issue, but apparently the AMD FirePro V4800
card which I got specifically for running SolidWorks and F360 was not supported
on Win10.

I decided to model the part anyway, even though my eyes were doing the shimmy
from the odd DPI scaling. I clicked on the F360 launcher, and just
as it came up, a modal notification box informed me that I was not
to unplug my machine because Windows was doing updates. Argggg

After waiting patiently for about 30min for it to reach 1% completion,
I unpluged the PC and booted back into Debian Linux.
About 5min later I had the part modeled in [SolveSpace](http://solvespace.com),
sliced in [Slic3r](http://slic3r.org/) and
printing on a machine running [Marlin](http://marlinfw.org/) firmware.

Guess I should boot into windows sometime when I don't need to use the PC and
let it finish those updates ...

SolveSpace and STL files are available on [Thingiverse](https://www.thingiverse.com/thing:31764671).

{% include image.html name="on_printer.jpg" %}

{% include image-gallery.html %}
