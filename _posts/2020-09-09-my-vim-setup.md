---
title: "My Vim setup"
tags:
- code
- vim
- configuration
excerpt: In which I discuss how I use Vim instead of an IDE to edit Java code.
---
## Background

People sometimes ask me about my development environment. This often happens after I give a presentation, or during a pair programming session. They ask because I'm using Vim. From a terminal, no less. Doing Java. What gives?

![Java in Vim](/images/2020-09-09-my-vim-setup/vim-1.png)

Many years ago, I used Eclipse for all of my Java editing. But it slowly became bloated and heavy, and around 2015 I switched to IntelliJ, which felt much faster and lighter. Unfortunately, to my tastes, IntelliJ has slowly been going in that same direction. Don't get me wrong, IntelliJ is still an amazing piece of software, and its slowness has a good reason: IntelliJ has some really amazing features that simply take a lot of computing power.

But it increasingly feels like it's no longer the IDE for me.

I've been a Vim user for a long time. I used it before I even started to use Eclipse. I like it because Vim is extremely fast and configurable. Unfortunately, for most of that long time, Vim was not a viable option for Java (or Scala) development. But something has changed in the last year or two: the [Language Server Protocol (LSP)](https://microsoft.github.io/language-server-protocol/) has become really good.

Honestly though, for most people, I still wouldn't recommend using Vim for Java development. It requires a lot of twiddling and fiddling to get it to work properly, and that takes time and dedication. For me, configuring Vim has become a hobby in its own right, and it must become for you as well if you desire to go in this direction. And even then, you won't have many of the features you're used to from IntelliJ.

In fact, you may be better off trying VS Code with its Java plugin (and maybe its Vim emulation plugin) if you're looking for a nice, "light-weight" alternative to IntelliJ. (I put "light-weight" between quotes because VS Code still uses Electron under the hood, so yeah.)

There's a big debate going on inside the Vim community, whether Vim can replace an IDE. There's an even bigger debate whether you should even _try_. My take on this is that Vim isn't, and shouldn't try to be, an IDE. I think that the shell is the IDE, and Vim is merely its editor. Vim is not its Git client or its database client. It is not its search tool. It should not try to compile code. It is not even its debugger. But I do think Vim should be a great editor, and great editing includes things like code completion and some rudimentary refactoring.

## How did I do it?

What I'm about to describe may change at any time. As I said, configuring Vim is a hobby of mine, and I'll probably keep tinkering with my configuration as long as I'm able to write code.

Also note that this is not a tutorial. I'm not going to explain step by step how to configure everything: that would take much too long. Instead, I will give pointers to plugins and leave you to figure out how to configure them in a way that you like best.

But here are some quick pointers to plugins and tools that I use to make editing Java in Vim, at least for me, an enjoyable experience.

### Plugin management

To begin with, I use [vim-plug](https://github.com/junegunn/vim-plug) to manage my Vim plugins. That includes [maintaining a lock file](https://www.simplethread.com/editor-plugins-belong-in-lock-file/) using its `PlugSnapshot` command.

### Completions

Next up, I use [CoC](https://github.com/neoclide/coc.nvim) to provide completions through the (LSP).

![Java with completions in Vim](/images/2020-09-09-my-vim-setup/vim-2.png)

I'm not completely happy with CoC, because it runs quite a heavy Node process in the background, it has its own plugin ecosystem outside of Vim's own ecosystem, and it can completely take over your Vim, to the point where it almost doesn't feel like Vim anymore. That said, it currently provides the best experience by far: it has the most complete implementation of the LSP and provides many quality-of-life features such as fuzzy searching in completions. It is also extremely configurable.

That said, I'm still keeping an eye on more minimalistic competing plugins such as [vim-lsp](https://github.com/prabirshrestha/vim-lsp), [vim-lsc](https://github.com/natebosch/vim-lsc), and the native LSP implementation that NeoVim is working on.

I use [coc-java](https://github.com/neoclide/coc-java) to provide the Java language server, which ironically is a stripped-down version of Eclipse. For Scala, I use [coc-metals](https://github.com/scalameta/coc-metals), which provides convenience around the amazing [Metals](https://scalameta.org/metals/) language server.

Note that I didn't install these using `:CocInstall`, as the documentation suggests. Instead, I use Vim-Plug to manage CoC's extensions, as described [here](https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions#use-vims-plugin-manager-for-coc-extension), so I can keep those versions locked down.

### Running code

To quickly run a unit test or a Java class with a `main` method, I've written a [Python script](https://github.com/jqno/dotfiles/blob/1e4ccbefc511662fe8bfe09080a3b4ee173dae53/scripts/runjava.py) that I can call from Vim, using a small [Vimscript wrapper](https://github.com/jqno/dotfiles/blob/1e4ccbefc511662fe8bfe09080a3b4ee173dae53/vim/plugin/runjava.vim). It runs almost completely outside of Maven, using it only to generate a classpath file. Using `javac`, it quickly compiles any Java files you have open in your Vim if they have changed, then runs your class using `java` or JUnit. It uses [Dispatch](https://github.com/tpope/vim-dispatch) to run all this asynchronously.

### Linting

I use [A.L.E.](https://github.com/dense-analysis/ale) to do linting in the background. I have set up CoC to report its linting issues through A.L.E., so that any errors and warnings can be presented to me in a unified way.

### Debugging

I'm one of those people who use print statements instead of a debugger, so this section is going to be quite lazy. I hear good things about [Vimspector](https://github.com/puremourning/vimspector) paired with [coc-java-debug](https://github.com/dansomething/coc-java-debug), though I haven't tried it yet.

### Navigation

When I said earlier that Vim is an editor, not an IDE, I left out the part where I do have a file browser plugin called [NERDTree](https://github.com/preservim/nerdtree). It's quite controversial in the Vim community, as many people think you shouldn't use a file explorer to open files in Vim; you should use a fuzzy finder like [FZF](https://github.com/junegunn/fzf.vim) instead. While I don't disagree, I do find it helpful sometimes to be able to see a tree outline of the files in my project. Java directory structures tend to be deep. I have it set up so that it's invisible by default and opens only when I hit a certain mapping.

![Scala with NERDTree in Vim](/images/2020-09-09-my-vim-setup/vim-3.png)

For navigating code semantically, [Vista](https://github.com/liuchengxu/vista.vim) is quite nice. It hooks into the LSP via CoC to display symbols from your code in a sidebar.

Finally, I use good old CTags (paired with [vim-gutentags](https://github.com/ludovicchabant/vim-gutentags) to generate them) because it's just so damn fast, and CoC's go-to-definition feature sometimes just doesn't work well for some reason.

### Refactorings

Some refactorings, like renaming, are provided by CoC. Some aren't. I have accumulated a bunch of Vim plugins that each do one specific type of refactoring. I encourage you to look around for the ones you're missing. The ones that I use include:

* [vim-commentary](https://github.com/tpope/vim-commentary)
* [vim-swap](https://github.com/machakann/vim-swap)
* [jqno-extractvariable.vim](https://github.com/jqno/jqno-extractvariable.vim) (written by me)

Still, this is one area where IntelliJ really shines and Vim often comes up short.

### Git

Again, I said before that Vim shouldn't be your Git client, and I still believe in that. But sometimes it's useful to do a quick git blame, and for that, no plugin is better than [vim-fugitive](https://github.com/tpope/vim-fugitive), even if it contains a huge amount of features that I never use.

### Color scheme

I have a weird color scheme I created myself, called [reversal.vim](https://github.com/jqno/reversal.vim). I find that most color schemes emphasize the reserved words in a language by giving them bright colours. Reversal.vim does the opposite, and emphasizes identifiers instead. After all, that's what the code is really about, right?

## Wrapping up

So, there you have it. You can check out my full Vim configuration [in my dotfiles repo](https://github.com/jqno/dotfiles/tree/main/vim). Please tweet me if you have ideas for improvement 😉.

