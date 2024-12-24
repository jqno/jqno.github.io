---
title: "Reviewing large PRs"
excerpt: In which I over-engineer a shell script.
tags:
- git
- code-review
- script
---
Do you have to do code reviews sometimes? I do. At my current project, we use GitHub PRs for that. Some PRs are small, and can easily be comprehended just by looking at them on the PR page, but some PRs are too large for this. The linear view that GitHub's PR page provides provides little help to fully understand the changes made in the PR. So what do you do?

You could power through and read it from the PR page anyway, which I don't recommend: it's very hard to keep the big picture in your mind and you'll end up pointing out only small things like typos. Or you can checkout its corresponding branch in your IDE and look at it there. This is better, because you can leverage your IDE to navigate through the code to find the big picture. However, you can't see the diff in the IDE, so it's hard to keep track of what code belongs to the PR and what doesn't. You might miss things, or look at things that aren't part of the PR.

I was in this situation often enough that I decided to write a script to help me deal with it. You can run it like this:

```sh
git review 1981
```

`1981` is the PR-number for the PR that you want to review. Let's say that its branch name is `feature/add-the-big-new-killer-feature`. When you run the script, it will create a _new_ branch called `review/feature/add-the-big-new-killer-feature`. This new branch will contain a bunch of new and/or changed files. These changes are _precisely_ the changes in the PR you are reviewing. So in essence you're seeing the branch in the same way that the PR author saw it, just before creating the commit (as if they had collapsed all changes into a single commit). You can now use your familiar IDE to navigate the PR, simply by navigating the changes in the branch that you're working on. If you want to suggest a change, you can even try it out and see if it actually works, before making the suggestion.

Why doesn't this script just open the branch `feature/add-the-big-new-killer-feature` and "uncommit" whatever is in there, you might ask? Well, this new branch gives you a clean playground which prevents you from accidentally committing and pushing something to the actual PR.

Also note that your working directory has to be clean for this script to run; otherwise it will exit early with an error message.

When you're done reviewing the PR, you can run `git review done` and the script will clean up any changes you made, delete the review branch, and return you to the branch you were in before you started the review.

Pretty nifty, right?

You can find the code for the script below this post. I've added comments so you can follow along with the logic; there's some pretty funky git-fu going on in there.

You can put the code in a file named `git-review`. The `-` is important: that way, `git` can pick it up and pretend that it's an actual `git` subcommand. Of course you can also give it another name if you prefer. Next, [put it in a directory that's on your path](https://apple.stackexchange.com/q/275343/175504).

Note that you need to have the [GitHub CLI](https://cli.github.com/) installed: the script uses it to determine the name of the branch for a PR. If you don't want to install it, you can modify the script to checkout branches instead of PRs. In that case, remove the block that checks if the GitHub CLI is available, and replace the line that says `pr_branch=$(gh ...)` with `pr_branch=$1`. That will allow you to run the script using `git review feature/add-the-big-new-killer-feature`.

And yes, I've been told that IntelliJ IDEA comes with very similar functionality out of the box. But I haven't been able to find it yet 😅. Please let me know if you know where it is, and how it compares!

```sh
#!/bin/bash

# Check if we're in a Git repo
if ! git rev-parse --is-inside-work-tree &>/dev/null; then
    echo "❌ Not in a git repository"
    exit 1
fi

# Check if the user provided a command-line argument
if [[ -z "$1" ]]; then
    echo "Usage: git review [<pr-number> | done]"
    exit 1
fi

if [[ "$1" == "done" ]]; then
    # We were reviewing; now we're done and we want to go back to where we were

    # Determine name of current branch
    current_branch=$(git rev-parse --abbrev-ref HEAD)

    # Check if we're on a review branch
    if ! [[ $current_branch =~ ^review/ ]]; then
        echo "❌ This is not a review branch"
        exit 1
    fi

    # Clean up the branch
    git reset --hard > /dev/null 2>&1
    git clean -fd > /dev/null 2>&1

    # Go back to the branch that was active before the review started
    git checkout - > /dev/null 2>&1

    # Remove review branch
    git branch -D "$current_branch" > /dev/null 2>&1

    echo "✅ Done reviewing $current_branch"

else
    # We want to start a review session

    pr_number=$1

    # Check if there's changes
    if git status --porcelain | grep -q '^'; then
        echo "❌ There are changes or untracked files in the repository"
        exit 1
    fi

    # Check if the GitHub CLI command is available
    if ! command -v gh > /dev/null 2>&1; then
        echo "❌ GitHub CLI not available"
        exit 1
    fi

    # Make sure we have everything
    git fetch

    # Determine the name of the PR's branch (using GitHub CLI)
    pr_branch=$(gh pr view "$pr_number" --json headRefName --jq '.headRefName')

    # Remove review branch if it exists already
    git branch -D "review/$pr_branch" > /dev/null 2>&1

    # Checkout the PR's branch into a new branch which mirror's the PR's branch, but with 'review/' in front
    git checkout -b "review/$pr_branch" "origin/$pr_branch" > /dev/null 2>&1

    # Fail if the branch doesn't exist
    if [[ $? != 0 ]]; then
        echo "❌ Branch doesn't exist (was the PR already merged?)"
        exit 1
    fi

    # Sever the link with the remote branch, so we don't accidentally push something to it
    git branch --unset-upstream "review/$pr_branch"

    # Determine name of main/master branch
    main_branch=$(git rev-parse --abbrev-ref origin/HEAD)

    # Find the point where the PR split off from main
    mergebase=$(git merge-base "$main_branch" "review/$pr_branch")

    # Check if this all worked correctly
    if [[ -z "$mergebase" ]]; then
        echo "❌ Could not determine a mergebase for branch $pr_branch"
        exit 1
    fi

    # Reset the changes made in the PR so we can look at them in our editor
    git reset --mixed "$mergebase" > /dev/null

    echo "🚀 Ready to review PR $pr_number from branch $pr_branch"
fi
```

> Cross-posted with permission. The original post is at [the Yoink blog](https://yoink.nl/posts/2024/12/24/reviewing-large-prs.html).
