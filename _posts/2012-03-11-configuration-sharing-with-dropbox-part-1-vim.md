---
title: "Configuration sharing with Dropbox, Part 1: VIM"
excerpt: Part 1 of my series, in which I show how I shared my .vimrc through Dropbox.
tags:
- configuration
- dropbox
- how-to
- vim
---
Introduction
------------

This article explains how to easily share your `.vimrc` _and_ all the plugins,
color schemes, and syntax highlighters that you use between all your
workstations.

If you haven't done so already, please read [Part 0]({% post_url 2012-03-11-configuration-sharing-with-dropbox-part-0-introduction %}) first. Go ahead, I'll wait.

One-time set-up
---------------

First, let's set up a VIM folder in your Dropbox:

* Create a folder called `vim` inside your `~/Dropbox/config`.
* Copy your existing `.vimrc` file to this folder, and rename it to `vimrc.vim`
  so the file doesn't get hidden if you're on OS X or Linux.
* Copy the entire contents of your `~./vimrc` folder into this new folder.<br>
  **Note!** If you use the [Command-T plugin](https://wincent.com/products/command-t),
  take care not to copy anything related to that. These are native,
  platform-dependent files, and can't be shared, unfortunately.
* Add a file called `vimrc.local` to the folder, with the following
  contents:

{% highlight vim %}
  if has('win32') || has('win64')
    set runtimepath^=$HOME/Dropbox/config/vim
    source ~\Dropbox\config\vim\vimrc.vim
  else
    set runtimepath^=~/Dropbox/config/vim
    source ~/Dropbox/config/vim/vimrc.vim
  endif
{% endhighlight %}

Per machine set-up
------------------

Next, for each of your computers, you should do the following things.

* First of all, if you want to use the excellent
  [Command-T plugin](https://wincent.com/products/command-t),
  go ahead and install that first. It needs to compile stuff to native code, so
  unfortunately, it's not portable across machines.
* Next, copy `config/vim/vimrc.local` from your Dropbox.  
    * On Linux and OS X, move it to `~/.vimrc`.
    * On Windows XP,move it to `C:\Documents and Settings\`_[your
      username]_`\_vimrc`.
    * On Windows 7, move it to `C:\Users\`_[your username]_`\_vimrc`.

* From now on, `~/Dropbox/config/vim/vimrc.vim` is your master `.vimrc` file.
  You can make all your changes there.
* If you find any external plugins, color schemes, or syntax highlighters on
  the [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki), just drop them
  into `~/Dropbox/config/vim` where otherwise you would have dropped them into
  `~/.vim`.

That's it!

