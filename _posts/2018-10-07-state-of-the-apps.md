---
title: "State of the Apps 2018"
excerpt: In which I make a list of stuff I have installed on my laptop, for future reference
tags:
- tools
---
State of the apps 2018

After the upgrade to MacOS Mojave borked my laptop the other day, I was forced to reinstall everything. It took me a while to find all my apps again, so I figured I'd make a list to make it easier next time. Hopefully that moment won't come any time soon of course, but maybe it will be fun to look at this list in 5 years and look at it with nostalgia over that one app that used to be great but isn't anymore, or with amazement over that one app that's still great after all this time.

# Getting shit done

* [**Airmail**](http://airmailapp.com) is a nice e-mail app with keyboard shortcuts, which supports multiple accounts which is nice. Don't forget to configure it: disable the Unified Inbox and make 'Selected Account' the default sender, or else you'll accidentally send mails from the wrong account. It's the only app in this list that I install through the App Store.

* [**Dropbox**](https://www.dropbox.com/register). For obvious reasons.

* [**MacPass**](https://macpassapp.org) currently seems to be the best KeePass client for MacOS. You are using a password manager, right? I use KeePass because it's open source (so everybody can theoretically verify there's no funny business) and it allows you to manage the password file yourself (cloud-based password managers tend to be the target of hacks a lot). I keep my file in Dropbox, which is ok because it's heavily encrypted with a password only I know.

* [**Rambox**](https://rambox.pro) is very because it allows you to keep all of your communications webapps in one window. I use it for Whatsapp Web and various Slack groups I'm in, but it supports many, many more. Before I switched to Rambox, I used [Franz](https://meetfranz.com) for a while. I forgot why I stopped liking it though.

# Browser

* [**Firefox**](https://www.mozilla.org/firefox). It's my main browser, because I prefer it over Chrome for some reason. I also use a few add-ons:

  * [**Privacy Badger**](https://www.eff.org/privacybadger) is a nice add blocker by the Electronic Frontier Foundation so you know it doesn't have a weird business model.

  * [**Impulse Blocker**](https://addons.mozilla.org/en-US/firefox/addon/impulse-blocker) is one of many site blockers. I don't have any preferences for any specific site blockers, but this one seems nice and simple. I use it to keep myself away from Facebook. If I really want to check Facebook (which occasionally happens -- I'm only human, after all), I force myself to go to a different browser, so Facebook can track me as little as possible.

  * [**KeePass**](https://addons.mozilla.org/en-US/firefox/addon/keepass-tusk) seems to be the most user-friendly KeePass plugin for Firefox. The others try to make some sort of connection with MacPass, which is very fiddly to set up.

* [**Chrome**](https://www.google.com/chrome). I use it when I can't or don't want to use Firefox for whatever reason. I'm logged into my secondary Google account there.

# Terminal

* [**Dotfiles**](https://github.com/jqno/dotfiles). Not really an app, but having all of my configuration files in an easy to install repository, means I don't have to recreate everything, which has saved me an enormous amount of time.

* [**iTerm2**](https://iterm2.com/downloads.html). I don't remember why I started using it instead of the regular Terminal app, but so many of my Hammerspoon scripts (see below) are based around it, that it just seemed easier to install iTerm.

* [**Homebrew**](https://brew.sh) is obviously the best way to install a whole bunch of command-line applications.
  * **`ZSH`** is my shell of choice. There's no need to install something like Oh-my-zsh, which I find gets in the way every time it tries to update. Everything that Oh-my-zsh offers is very easy to add by hand: there's Homebrew packages for certain plugins like syntax highlighting, and prompts don't need Oh-my-zsh either. Check my [dotfiles](https://github.com/jqno/dotfiles/tree/master/zsh) if you don't believe me.<br>`brew install zsh zsh-autosuggestions zsh-syntax-highlighting`
  * **`git`**, **ctags** and **`rsync`** I install with Homebrew because the built-in versions are outdated.<br>`brew install git ctags rsync`
  * **`ag`** (or **The Silver Searcher**) is a replacement for `grep` that I find easier to work with. It's also (supposed to be) faster.<br>`brew install ag`
  * **`bat`** is a replacement for `cat` that does syntax highlighting. I was trying it out before my laptop broke so I might not keep it, but so far it seems nice.<br>`brew install bat`
  * **`fzf`** is a fuzzy finding tool for the command-line. It's great for finding stuff in your shell history. It's also great in Vim to find files, buffers, tags, and other things.<br>`brew install fzf`, and don't forget to also run the install script for keybindings.
  * **`pandoc`** is a tool that converts documents from one format to another. I use it when people expect me to write something in Word; I just convert it from Markdown with Pandoc. Also, it's pretty nice for converting Markdown to Reveal.js presentations.<br>`brew install pandoc`
  * **`tig`** is a very nice terminal-based app to view logs and diffs in Git.<br>`brew install tig`
  * **`vim`** because I also use it in the terminal.<br>`brew install vim`

# Programming

* [**IntelliJ**](https://www.jetbrains.com/idea/download) for Java and Scala development. Let's also install the Scala plugin, IdeaVim and, yes, the [Nyan Progress Bar](https://plugins.jetbrains.com/plugin/8575-nyan-progress-bar) plugin.

* [**MacVim**](http://macvim-dev.github.io/macvim/) because I'm one of _those_ people. I download it from the website instead of through Homebrew, because MacVims managed by Homebrew tend to break for me a lot for some reason.

* [**GitUp**](https://gitup.co) is a super handy tool to fix and re-write your Git history. Use it at your own risk. 😉

# Customization

* [**Alfred**](https://www.alfredapp.com) is great as a replacement for Spotlight, but these days I use it even more for its snippet engine, which lets me quickly type emoji, [special characters](http://jqno.nl/ComposeKey.alfredsnippets), and ASCII art like ¯\\\_(ツ)\_/¯ and ಠ_ಠ. See also my post about [an even better way of typing]({% post_url 2017-12-06-an-even-better-way-of-typing %}).

* [**Bartender**](https://www.macbartender.com). When you install as many tools as I do, it's nice to be able to clean up your menu bar. As a bonus, it also allows you to hide the clock without completely getting rid of it, which gives me a nice feeling of zen when I work.

* [**Hammerspoon**](https://www.hammerspoon.org) is a tool that allows you to script almost any aspect of MacOS. I use it as a window manager, for quickly opening a terminal and switching between apps, and many more things. Just check my [dotfiles](https://github.com/jqno/dotfiles/tree/master/hammerspoon) for examples.

* [**Karabiner Elements**](https://pqrs.org/osx/karabiner) for redefining some of my keyboard's keys. I use it mainly to switch the `` ` `` and `~` keys, which for some reason are swapped on Dutch Macbooks. I also use it to remap CapsLock with a makeshift Hyper key, which I use heavily in my Hammerspoon scripts to define global shortcuts.

* Bonus app: [**Night Owl**](https://nightowl.kramser.xyz) is a new tool for Mojave that switches between Light Mode and Dark Mode based on the time of the day. Nice!

# Miscellaneous

* [**Jekyll**](https://jekyllrb.com) is what I use to build this website.

* [**Lightroom**](https://account.adobe.com/orders). I use it to catalog my photos. I have a single license for it, back from when it was still possible to get one. Since it's becoming increasingly hard to find the correct download link (Adoby are pushing their Creative Clound subscription service HARD), I include it here.

* [**Spotify**](https://www.spotify.com/download/mac). Gotta have my tunes.

* [**MS Office ಠ_ಠ**](https://www.office.com) because yes, even I need that from time to time. Usually when someone sends me a screenshot embedded in a Word file or something.

