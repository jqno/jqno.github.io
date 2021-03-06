---
title: "Configuration sharing with Dropbox, Part 3: Bash"
excerpt: Part 3 of my series, in which I show how I shared my .bashrc and other bash scripts through Dropbox.
tags:
- bash
- configuration
- dropbox
- how-to
---
Introduction
------------

In this article, I will explain how you can easily share shell scripts between your workstations. Some of these scripts, such as convenient aliases or a [git color prompt script]({% post_url 2012-04-02-howto-display-the-current-git-branch-in-your-prompt %}), have to be run each time you start your shell; others only have to be accessible from the PATH. Both cases are covered by this trick.

But first, you should read [Part 0]({% post_url 2012-03-11-configuration-sharing-with-dropbox-part-0-introduction %}), if you haven't done so already.

One-time set-up
---------------

* Create a folder called `bash` inside your `~/Dropbox/config`.
* Create two subfolders: `scripts` and `session`.<br>`scripts` will contain the
  scripts that need to be on the PATH; `session` will contain the scripts that
  have to run each time you open up a terminal.
* Create the `run.sh` file in `~/Dropbox/config/bash`. (Indeed; it shouldn't be
  in `scripts` or `session`!) Make it executable. Its contents should be as
  follows:

{% highlight bash %}
  # Add the following line to ~/.bashrc:
  # . /path/to/this/script/run.sh script_folder
  # For example:
  # . ~/Dropbox/config/bash/run.sh session
  # Don't forget the dot!

  for f in `dirname ${BASH_SOURCE[0]}`/$1/*; do
    . $f
  done
{% endhighlight %}

  This file will iterate over all the files in the given directory; in this case, `~/Dropbox/config/bash/session/*`, and execute them.

* Finally, create a file called `add-scripts-to-path.sh` in the
  `~/Dropbox/config/bash/session` folder. Again, don't forget to make it
  executable. It should contain the following one-liner:

  {% highlight bash %}export PATH=$PATH:\`dirname ${BASH_SOURCE[0]}\`/../scripts{% endhighlight %}

  ... which is how `~/Dropbox/config/bash/scripts/` ends up on the PATH. 

Per computer set-up
-------------------

On each computer, you need to add the following line to the end of your
`~/.bashrc` to make it all work:

{% highlight bash %}. ~/Dropbox/config/bash/run.sh session{% endhighlight %}

Don't forget the dot! It won't work without it. And that's all!

