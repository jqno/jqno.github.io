---
title: "Configuration sharing with Dropbox, Part 0: Introduction"
tags:
- configuration
- dropbox
- how-to
time: "12:00"
---
If you're anything like me, you have more than one pc that you regularly use.
One at home, one at work, maybe more. If you're even more like me, there is
software that you use on each of these pc's. VIM, Eclipse, maybe more. And,
exactly like me, you've configured these programs heavily to suit your every
fancy. And you want to share these configurations across all your pc's, with a
minimum of hassle. And, of course, if you re-install your pc, you want to
restore your configurations with no effort.

What are you, a programmer or something?

Of course you don't want to e-mail files to yourself or push and pull stuff on
GitHub. I mean, that works, but it's too easy to forget, and <span
style="text-decoration: line-through;">you're</span>I'm too lazy to bother
anyway. Changes should propagate automatically!

This is Part 0 in a series of posts that will help you do just that, using
Dropbox. In each of the next installments, I will highlight one specific
program and explain how to share its configuration in such a way that you can
easily tweak your configuration and have it pushed to each of your workstations
automatically.

This series of posts is based on a [StackOverflow answer](http://stackoverflow.com/a/1184897/127863) I once gave. I've expanded a
bit on it since then, and I'm documenting that expansion here.

Set-up
------

This post explains the preliminaries of my system. Before you move on to one of
the later parts in this series, you'll need to follow these steps:

* Download and install [Dropbox](http://www.dropbox.com) on each of your
  workstations.
* Create a folder called `config` inside your
  `~/Dropbox`<sup>[[1]](#note1)</sup>
  folder.

That's it.

Further reading
---------------

Now, you can open any of the following posts to find out how to set it up for your favourite software<sup>[[2]](#note2)</sup>:

* [Part 1: VIM]({{ pcposturl(2012, 03, 11, "configuration-sharing-with-dropbox-part-1-vim") }})
* [Part 2: Eclipse]({{ pcposturl(2012, 03, 17, "configuration-sharing-with-dropbox-part-2-eclipse") }})
* [Part 3: Bash]({{ pcposturl(2012, 03, 24, "configuration-sharing-with-dropbox-part-3-bash") }})
* Part 4.._n_: other cool software _(hoped for)_

Enjoy!

Footnotes
---------

<a name="note1"></a>[1] Note that in this post, and all subsequent ones, I'll be assuming Unix file conventions, unless noted otherwise. So if you're on Windows, replace all `/`'s with `\`'s, and replace `~/Dropbox` with `C:\Documents and Settings\`_[your username]_`\My Documents\My Dropbox`.

<a name="note2"></a>[2] Well, actually it's _my_ favourite software. But maybe yours is in this list, too!

