---
layout: post
title:  Colored Formatter for Python's Logging Module
date:   2017-09-29 04:56:25 +0500
categories: Programing
tags: Python
description: A custom colored formatter for Python's logging module.
---

This is nothing fancy, but it is very simple to use and does not change the
record object, thereby avoids logging the ANSI escape sequences to a log file
if a file handler is used. It does not effect the log message formatting.

If you are already using the logging module's
[Formatter](https://docs.python.org/3/library/logging.html#formatter-objects),
all you have to do to get colored level names is to replace your counsel handlers
Formatter with the ColoredFormatter. If you are logging an entire app you only 
need to do this for the top level logger.


<figure class="lineno-container">
<figcaption>colored_formatter.py</figcaption>
{% highlight python linenos %}
#!/usr/bin/env python

from copy import copy
from logging import Formatter

MAPPING = {
    'DEBUG'   : 37, # white
    'INFO'    : 36, # cyan
    'WARNING' : 33, # yellow
    'ERROR'   : 31, # red
    'CRITICAL': 41, # white on red bg
}

PREFIX = '\033['
SUFFIX = '\033[0m'

class ColoredFormatter(Formatter):

    def __init__(self, patern):
        Formatter.__init__(self, patern)

    def format(self, record):
        colored_record = copy(record)
        levelname = colored_record.levelname
        seq = MAPPING.get(levelname, 37) # default white
        colored_levelname = ('{0}{1}m{2}{3}') \
            .format(PREFIX, seq, levelname, SUFFIX)
        colored_record.levelname = colored_levelname
        return Formatter.format(self, colored_record)
{% endhighlight %}
</figure>

## Example usage

<figure class="lineno-container">
<figcaption>app.py</figcaption>
{% highlight python linenos %}
#!/usr/bin/env python

import logging
from colored_log import ColoredFormatter

# Create top level logger
log = logging.getLogger("main")

# Add console handler using our custom ColoredFormatter
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
cf = ColoredFormatter("[%(name)s][%(levelname)s]  %(message)s (%(filename)s:%(lineno)d)")
ch.setFormatter(cf)
log.addHandler(ch)

# Add file handler
fh = logging.FileHandler('app.log')
fh.setLevel(logging.DEBUG)
ff = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(ff)
log.addHandler(fh)

# Set log level
log.setLevel(logging.DEBUG)

# Log some stuff
log.debug("app has started")
log.info("Logging to 'app.log' in the script dir")
log.warning("This is my last warning, take heed")
log.error("This is an error")
log.critical("He's dead, Jim")

# Import a sub-module 
import sub_module
{% endhighlight %}
</figure>

<figure class="lineno-container">
<figcaption>sub_module.py</figcaption>
{% highlight python linenos %}

#!/usr/bin/env python

import logging
log = logging.getLogger('main.sub_module')

log.debug("Hello from the sub module")
{% endhighlight %}
</figure>

## Results

<figure>
<figcaption>Terminal output</figcaption>
<img src="https://i.stack.imgur.com/OsmlG.png" alt="Terminal output">
</figure>

<figure class="lineno-container">
<figcaption>app.log content</figcaption>
{% highlight plain linenos %}
2017-09-29 00:32:23,434 - main - DEBUG - app has started
2017-09-29 00:32:23,434 - main - INFO - Logging to 'app.log' in the script dir
2017-09-29 00:32:23,435 - main - WARNING - This is my last warning, take heed
2017-09-29 00:32:23,435 - main - ERROR - This is an error
2017-09-29 00:32:23,435 - main - CRITICAL - He's dead, Jim
2017-09-29 00:32:23,435 - main.sub_module - ERROR - Hello from the sub module
{% endhighlight %}
</figure>

Of course you can get as fancy as you want with formatting the terminal and log 
file outputs. Only the log level will be colorized.

The Python example files can be downloaded from the GitHub Gist located 
[here](https://gist.github.com/KurtJacobson/48e750701acec40c7161b5a2f79e6bfd).
