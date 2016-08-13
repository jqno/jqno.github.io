---
title: "How to: Display the current Git branch in your prompt"
tags:
- bash
- git
- how-to
- prompt
---
Git is the best version control system I've worked with (so far). I started
feeling this way once I got the hang of branching. But if you have a lot of
branches lying around, it's easy to get lost. Fortunately, it's easy to fix
that problem, by pimping your terminal prompt. Mine now looks like this:

![Git prompt](/images/2012-04-02-howto-display-the-current-git-branch-in-your-prompt/git-prompt.png)

And suddenly, keeping track of branches has become easy.

There are many blog posts floating around the internet that explain how to make
your prompt look like this, and this post is but one of them. What makes mine
different, is that it has the following features:

* It works on both Ubuntu and OS X.<br>The first git prompt that I used, turned
  out not to work on OS X for some reason. This one does.
* Dirty flag<br>If you've made a change, a * will be added to the name of the
  branch, showing you have outgoing changes.
* Merge conflicts flag<br>If you have merge conflicts, a big yellow |MERGING
  will be added to the name of the branch.
* A no email setting flag<br>I have no global user.email setting, because I
  want to push to work repos and my personal GitHub from the same machine, and
  not accidentally use the wrong email address. However, occasionally I would
  forget to set my email in a new repository, and push garbage to it. No more!
  This flag will remind me.
* Colors to recognise different pieces of information at a single
  glance<br/>Although it does kind of look like a christmas tree this way.
  Fortunately, the colors are easily changed.

Here's the code for my prompt. Save it as `~/fancy-prompt.sh` and call it from
your `~/.bashrc` like this:

{% highlight bash %}
. ~/fancy-prompt.sh
{% endhighlight %}

The dot at the start of the line is important; it won't work if you leave it out!

Or better yet: add it to [your Dropbox]({% post_url 2012-03-24-configuration-sharing-with-dropbox-part-3-bash %})!

{% highlight bash %}
function _fancy_prompt {
  local RED="\[\033[01;31m\]"
  local GREEN="\[\033[01;32m\]"
  local YELLOW="\[\033[01;33m\]"
  local BLUE="\[\033[01;34m\]"
  local WHITE="\[\033[00m\]"

  local PROMPT=""

  # Working directory
  PROMPT=$PROMPT"$GREEN\w"

  # Git-specific
  local GIT_STATUS=$(git status 2> /dev/null)
  if [ -n "$GIT_STATUS" ] # Are we in a git directory?
  then
    # Open paren
    PROMPT=$PROMPT" $RED("

    # Branch
    PROMPT=$PROMPT$(git branch --no-color 2> /dev/null | sed -e "/^[^*]/d" -e "s/* \(.*\)/\1/")

    # Warnings
    PROMPT=$PROMPT$YELLOW

    # Merging
    echo $GIT_STATUS | grep "Unmerged paths" > /dev/null 2>&1
    if [ "$?" -eq "0" ]
    then
      PROMPT=$PROMPT"|MERGING"
    fi

    # Dirty flag
    echo $GIT_STATUS | grep "nothing to commit" > /dev/null 2>&1
    if [ "$?" -eq 0 ]
    then
      PROMPT=$PROMPT
    else
      PROMPT=$PROMPT"*"
    fi

    # Warning for no email setting
    git config user.email | grep @ > /dev/null 2>&1
    if [ "$?" -ne 0 ]
    then
      PROMPT=$PROMPT" !!! NO EMAIL SET !!!"
    fi

    # Closing paren
    PROMPT=$PROMPT"$RED)"
  fi

  # Final $ symbol
  PROMPT=$PROMPT"$BLUE\$$WHITE "

  export PS1=$PROMPT
}

export PROMPT_COMMAND="_fancy_prompt"
{% endhighlight %}
