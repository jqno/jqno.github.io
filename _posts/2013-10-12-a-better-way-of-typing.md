---
title: "A better way of typing"
tags:
- compose-key
- os-x
- windows
- linux
- how-to
---
I'm happy! I'm happy because I just found out how to get a Compose key on my shiny MacBook. I want you to be happy too, so I'll show you how you can get one, too. I'll even show you how to do it on Windows and Ubuntu. But first, I will explain why this makes me so happy. (If you don't care about the why, just click [here](#How) and go straight to the how. But you'll miss a perfectly good geek-out.)

Why?
====

I'm a language geek. I like to spell words correctly. Words such as _naïveté_. Actually, I'm also Dutch, so I have to deal with words like _coëfficiënt_ and of course _hé_ and _hè_ (which, really, are two very distinct words). I speak French (or should I say: _français_) and I live in a country where we pay with _€_. And if that's not enough, I like to spell people's names correctly. I'm a software engineer; we're a pretty international bunch. I've worked with people called _René_, _Radovanović_, and even _Enikő_. Yes, that's an _o_ with a double accent aigu. It's Hungarian. Try finding that key on your keyboard.

I want it to be **easy** to type these special characters. I also want it to **not get in the way**.

Windows has this nifty trick. If you use the US International keyboard layout, you can press a `"` followed by an `e` and you automatically get an _ë_. Sounds nice in theory, but for me, it gets in the way. I'm a programmer; I need to be able to type things like `String vowel = "e";`. And when I do, I don't want that to show up like `String vowel = ë";`. That's just annoying. On a day to day basis, I have to type `"e` much more often than I have to type `ë`.

Of course, if I type a space after I type `"`, I get my precious `"`, and I can then type an `e` and it won't turn into an _ë_ anymore. I know people, programmers like me even, who have this [extra key stroke](http://www.keysleft.com/) ingrained in their muscle memory. If they work on a computer where this option is disabled, they type things like `String vowel = " e";`. Notice the extra space? Now I have nothing against these people, but this is just plain stupid. It's like hitting your face every five minutes hoping to catch a fly.

OS X has something slightly smarter. To get a special character, you press a special key combination. For example, to get an _é_, you press `⌥E`, followed by `e`. If you want an _ë_, you press `⌥U`, followed by `e`. The problem with those `⌥` combinations is that they're pretty arbitrary: `e` for `´`, `u` for `¨`, and `i` for `ˆ`. In fact, I can never remember what key corresponds to what symbol. It's not easy.

Also, both methods only support a very limited set of special symbols. Enikő is out of luck; she has to hunt through the Character Map program to spell her name on a non-Hungarian keyboard. If you think that's too exotic, then you should realise that _€_ isn't directly supported on many systems, either.

I'm not even going to mention Alt codes.

Surprisingly, Linux, for all its usability-issues, has an easy and elegant solution to this mess: the [Compose key](http://en.wikipedia.org/wiki/Compose_key). You pick a key you don't use often (I like the Menu key: nobody uses that button anyway) which becomes the Compose key. If you want to enter a special symbol, you hit this button, followed by the two (or more) symbols that you want to 'compose'. For example, `Compose` followed by `"` followed by `e` becomes _ë_. `Compose` followed by 
`=` followed by `o` becomes _ő_, and `Compose` followed by `=` followed by `e` becomes _€_. Easy!

And it goes even further than that. Combine `-` and `>`, and you get an arrow: _→_, `T` and `M` become _™_, `<` and `3` become _♥_, and, I was surprised to find out while researching this article, `Compose`-`C`-`C`-`C`-`P` becomes _☭_. Seriously.

I like this. It doesn't get in the way at all. I'm free to type `String vowel = "e";` without any _ë_'s showing up. Also, it's super easy. In fact, it's so easy that sometimes when I'm bored, I amuse myself by trying out various combinations of keys to see what comes out. How would you type _Æ_, _¿_ or _©_?

By now, I have probably convinced you that you want to have a Compose key, too. So how do you get one? It's easy.

How?
====

<a name="How"></a>

* OS X: Install the [US custom](http://uscustom.sourceforge.net/) keyboard layout: download and installation instructions are [here](http://uscustom.sourceforge.net/#installation).<br>Note that MacBooks don't have a Menu key. This tool uses the `§` key instead, which is fine, because who uses it anyway? And even if you do, it's a `Compose`-`s`-`o` away.
* Windows: Install [this nifty little program](https://github.com/SamHocevar/wincompose).
* Ubuntu: Go to System Settings → Keyboard Layout → Options → Compose key position.
* Any other Linux? Then you probably already know how to do this.

Enjoy a better way of typing! ☺
