---
title: "Using ChatGPT to write a complete app"
tags:
  - foobal
  - balgpt
  - chatgpt
  - ai
  - software-development
excerpt: "In which I share my experiences building an app with ChatGPT so I can start annoying my uncle again."
---
When I first started playing with ChatGPT, I had it write a [simple, low effort blog post]({% post_url 2023-01-25-my-experience-with-chatgpt-as-a-developer-impressed-but-with-caveats %}) to see what it could do (and [a follow-up using the same prompts]({% post_url 2023-03-15-my-adventure-with-chatgpt-a-developers-perspective %}) when GPT-4 came out). Those posts were a bit lazy. This is going to be a more serious exploration, written manually (but reviewed by ChatGPT 😉).

I wanted to write a full app with ChatGPT, just to see how far I could take it. I was still using Foobal, the soccer match predictor I use to play a pool with my relatives, which [I've]({% post_url 2012-09-11-foobal-predicting-soccer-matches-with-scala-and-drools %}) [also]({% post_url 2012-12-31-foobal-end-of-year-update %}) [blogged]({% post_url 2013-08-05-foobal-a-new-hope %}) [about]({% post_url 2014-08-04-on-profiling %}) [before]({% post_url 2015-06-11-foobal-moneyball %}). I never really maintained that app, while the ecosystem had progressed, so I have a hard time even building the thing these days. Also, it only has a CLI, and I want a web app. Time for an upgrade!

## Getting Started

I decided on the following tech stack:

- Go for the backend. I dislike Go, mainly because of how it forces you to mix error handling with business logic, instead of allowing you to separate the concerns. But I wanted something that produces native binaries, and it seemed easier to troubleshoot than Rust.
- Svelte for the frontend. It seemed like a nice, minimalistic framework.
- Postgres for the database. It's free, it's reliable, it provides everything I need.
- Fly.io for deployment. Infra is a weak point for me, and it promises to take that away for you, while still having a free tier. And it supports Postgres.

The plan was to have ChatGPT write everything, and not write any code myself. For the most part, that worked fine. As ChatGPT conversations grew, though, it tended to start hallucinating, assuming that certain bits of code were written a certain way when they weren't; and as the program grew, it became harder for ChatGPT to maintain everything in its context. So I sometimes had to start new conversations and paste in relevant bits of context, and manually glue them together. But I never had to hand-write anything big.

## Backend

Go turned out to be quite pleasant to work with, though I never had to write any error handling myself. I might think differently if I had had to. Also, Docker containers are absolutely tiny! I love that!

Scraping [the match results website](https://www.fcupdate.nl/voetbalcompetities/nederland/keuken-kampioen-divisie/programma-uitslagen), storing everything in the database, implementing a prediction algorithm: it all went very well.

Setting up infra with Fly.io was a breeze, too. It's well documented, and ChatGPT has obviously been trained with that documentation as well. That was a real confidence boost for me.

## Frontend

The frontend did not go so well at first. Setting up Svelte went fine, but when I started to do authentication using Google, things started going badly. Apparently Google had changed its API for that after ChatGPT's training cutoff and I could not get it to work with Svelte. I decided to throw out Svelte and go with raw HTML and JavaScript instead, and for a while that seemed to work, but after some more testing it turned out that Google auth still didn't work well, so I threw that out as well and opted for [Basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication). It's not as secure, but at least it works, and it's simple. I'll definitely have to revisit this later.

Another thing to revisit, is Svelte. I realize now that Svelte's authentication documentation wasn't the problem; it was Google's. While my app only has 2 pages (one for login and one for the predictions themselves), I already encountered some scaling issues where I want to share bits of code but can't really do so. So at least the experience has taught me the value of JavaScript frameworks, and I'd like to try again with Svelte, as it seems to be the most minimalistic one. Something like Angular is way too heavy for my two screens.

By the way, when I finally had a working frontend, I asked ChatGPT to "make it pretty". You can see the result below. There definitely still a place for designers in the new, AI-enhanced world!

![A not-very-pretty user interface](/images/2023-05-09-using-chatgpt-to-write-a-complete-app/pretty.png)

## Don't trust ChatGPT

The most important thing I've learned going this project, is never to trust any code that ChatGPT produces. That feels ironic, when building a complete app with it, and certainly during the initial hype of GPT-4 I didn't appreciate that enough. But as I mentioned before, ChatGPT often hallucinates code that simply doesn't work. Often it's easy enough to notice, for example when it produces something that simply doesn't compile with the existing codebase. But sometimes the bugs are more subtle, for example when I had it generate authentication: it looked right, I was able to log in to my app, so I didn't properly test it and continued with other parts of the app, until I noticed that it didn't actually check credentials and would log me in with incorrect credentials too. Oops. That was when I had to redo the whole thing using Basic authentication.

That also means I should probably have ChatGPT generate me some unit tests, because right now, I have none 🙊

Another thing I found, was that debugging an app written by ChatGPT is kind of hard, simply because I didn't write any of the code myself and sometimes I simply didn't know how it worked 😅

And finally, a lot of refactoring is also needed. ChatGPT doesn't always generate the simplest or cleanest code. Especially when you have to start a new conversation and it doesn't know all of the context anymore, because then you have to glue bits together that don't always fit perfectly.

Of course, these things apply for manually written code as well, but when using AI it's easy to forget. After all, what could go wrong with code that's generated by a machine? If there's a problem with it, the machine can fix it, right? Well, no, because a fix generated by ChatGPT might change existing behavior, so you still need tests. Code quality is still important, at least with the current generation of AI tools.

## Conclusion

All in all, it was a pleasant experience, and I was surprised at how fast I got something in production. I definitely feel a lot more productive using ChatGPT than I do without. I've since used ChatGPT for other projects too, that I probably wouldn't even have started otherwise, such as [a GNOME extension](https://extensions.gnome.org/extension/5696/one-window-wonderland) (I have another one planned). It has saved me from crawling Linux man pages, because it will just generate the right command line for me based on a badly specified prompt. And it's absolutely great at critiquing and improving texts, such as this one.

I'm happy with my tech stack. Go was nicer than I expected; Postgres has been reliable as always; Fly.io was an absolute delight! And I really need to give Svelte another chance, because I didn't give it enough of one.

ChatGPT itself was impressive, even if you always have to double-check what it gives you. It's a great enabler for doing things you aren't familiar with yet.

And I've successfully been doing soccer predictions using balGPT for a few weeks now. It's not going to be giving better predictions than Foobal was, because in the end they both use the same algorithms, but at least I'm now able to do it from my phone, because of my shiny new web interface.

All the code is on [GitHub](https://github.com/jqno/balGPT) if you're interested, and I've stored [all the prompts](https://github.com/jqno/balGPT/tree/main/prompts) in the GitHub repo, so you can check those out for yourself as well.

## PS

When I asked ChatGPT to fix the capitalizations in my final draft, it hallucinated this second conclusion for me. What do you think? Should I have kept it? 😉

> In conclusion, while ChatGPT has its limitations, it can still be an incredibly valuable tool in speeding up development, exploring unfamiliar technologies, and even getting creative input. With a critical eye and proper testing, it's possible to build a complete app with AI assistance. This experience has shown me the potential of AI in the world of software development and has inspired me to continue exploring and pushing the boundaries of what AI tools like ChatGPT can achieve.
>
> As the field of AI continues to develop, it will be exciting to see how these tools evolve, and how they can be further integrated into our workflows to make us even more productive and efficient. The future of AI-assisted development is bright, and I look forward to the advancements and discoveries yet to come.
