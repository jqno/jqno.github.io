---
title: "About fonts"
tags:
- font
- typography
- blog
excerpt: In which I fall into a typographic rabbit hole
---
I fell into a bit of a rabbit hole after reading Matthew Butterick's [Practical Typography](https://practicaltypography.com/): I got a little obsessed with fonts for a while. It convinced me that I want my website to look pretty, and one important part of that is the choice of font. There are three applications of fonts that have been on my mind: headings and body text on my website, and a coding font for my editor (and also for code snippets on my website). Let's look at each in turn, but first, let's set some criteria.

## Criteria for a font

The purpose of a font is to make text look good.

That's it. That's the criterion.

However, this is hard to determine. Of course, there's no accounting for taste, but Butterick talks about a lot of fonts in his book, and mentions which ones are good and which ones [aren't](https://practicaltypography.com/system-fonts.html). However, he doesn't give a lot of guidance on how to see the difference. Blogs on the web claim to do so, but don't go into a lot of depth and generally constrains themselves to free fonts.

Which leads to another factor: cost. According to Butterick, _"[most free fonts are garbage](https://practicaltypography.com/free-fonts.html)"_. I don't mind spending some money, but fonts can get really expensive. Butterick also designs fonts, and his seem to be relatively cheap. Still, it's a lot of money for a website that doesn't get a lot of traffic.

But sometimes, big-name companies such as [IBM](https://github.com/IBM/plex), [The Smithsonian](https://www.cooperhewitt.org/open-source-at-cooper-hewitt/cooper-hewitt-the-typeface-by-chester-jenkins/), Adobe [1](https://github.com/adobe-fonts/source-serif), [2](https://github.com/adobe-fonts/source-sans) and [Mozilla](https://github.com/mozilla/Fira) release good fonts (that Butterick even approves of) and that are designed by renowned professionals, which is nice! In these cases though, for ideological reasons, I do prefer companies that have somewhat of a proven track record in free and open source software. For instance, all else being equal, I'd pick Mozilla's Fira over IBM Plex.

Coding fonts have some other requirements as well. First of all, they need to be monospaced (well, for most people anyway), and they need to make the difference between certain symbols really clear: `0O`, `1lI`

## Coding

So let's talk about coding fonts first.

![Examples of coding fonts that I considered](/images/2024-01-10-about-fonts/coding.png)

The rabbit hole opened up in front of me when I read the section on [ligatures in programming fonts](https://practicaltypography.com/ligatures-in-programming-fonts-hell-no.html) in Butterick's book. He makes a few arguments for why they're a bad idea, though I don't think I ever experienced these problems myself. I've been using [Fira Code](https://github.com/tonsky/FiraCode) (a fork of Mozilla's Fira Mono with added ligatures) for a long while, and I have to say that I rather enjoy the ligatures.

But it did get me thinking, because I did get confused reactions sometimes from people looking at my screen, or at code snippets on my website or in [my talks](https://jqno.nl/talks/).

Also, Fira Code doesn't have italics, so the font configuration in my terminal was kind of complex, pulling in a separate font for italics.

So I started looking around. I tried [Julia Mono](https://juliamono.netlify.app/) for a while, but the letters felt uneven to me. I looked at [Gintronic](https://markfromberg.com/projects/gintronic) which I think looks cute, but I wasn't quite ready to drop $150 on it and risk getting bored with it later. I've also looked at [IBM Plex Mono](https://www.ibm.com/plex/), which is free and open source, but at the same time designed by typography professionals. But it left me feeling meh. I also wanted to try [Recursive Mono](https://www.recursive.design/), but I couldn't get it to work in my terminal, [Kitty](https://sw.kovidgoyal.net/kitty/). Your mileage may vary, though.

Then I discovered GitHub's new [Monaspace](https://monaspace.githubnext.com/) font. I specifically liked its Argon variant. It has a very cool "texture healing" feature, which moves wide letters like `m` and narrow letters like `i` a few pixels to the side to make the text look more evenly spaced. But it's [not supported on my terminal](https://github.com/githubnext/monaspace/issues/15). They've made a fix and promised to release it "soon", but it's been almost two months.

Eventually, I found [Commit Mono](https://commitmono.com/), a relatively new open source font that also does texture healing (they call it "smart kerning"). It has a few basic ligatures, which I've disabled. It looks really nice, but you do notice the letters "jumping around" a little while you type because of the smart kerning feature. I'm currently trying it out to see if I mind the jumping around, but I do like the overall looks of it. And it has a nice italic! I've already integrated it into my website.

Here's an example of Commit Mono's smart kerning feature at work. It's especially visible around the `m`s, which seem to have a lot more space:

![Example of Commit Mono's smart kerning](/images/2024-01-10-about-fonts/smart-kerning.png)


## Body text

I'd never really considered the font of my website's [body text](https://practicaltypography.com/body-text.html) before. The theme I use sets it to this by default:

```css
-apple-system, BlinkMacSystemFont, "Roboto", "Segoe UI",
    "Helvetica Neue", "Lucida Grande", Arial, sans-serif
```

In other words, it selects whatever default font is installed on the system. This means two things:

1. My site will look different on different devices, and
2. My site will look like every other site, and therefore boring.

As Butterick says, _"[like Times New Roman, Arial is permanently associated with the work of people who will never care about typography](https://practicaltypography.com/system-fonts.html)"_

![Examples of body text fonts that I considered](/images/2024-01-10-about-fonts/body-text.png)

So I thought it would be nice to change it up a little, to give my site that little extra _je ne sais quoi_, to make it stand out from the pack without readers necessarily noticing why.

I mainly looked at sans-serif fonts, because I think these look nicer on screen. I did consider Butterick's own font [Heliotrope](https://practicaltypography.com/heliotrope.html) for a bit, but it has a slight weirdness to it which I decided doesn't fit with my heading font of choice (see below).

Some sans-serif fonts I looked at were [Cooper Hewitt](https://www.cooperhewitt.org/open-source-at-cooper-hewitt/cooper-hewitt-the-typeface-by-chester-jenkins/) and [IBM Plex Sans](https://github.com/IBM/plex), but I didn't really like how they look.

In the end, I landed on Fira Code's cousin [FiraGO](https://github.com/bBoxType/FiraGO), the successor to Mozilla's [Fira Sans](https://github.com/mozilla/Fira). I think it looks nice! I particularly like the little opening in the bottom half of the g.

And you're looking at it right now!

## Headings

Headings should look different from the body text, because their point is to reveal structure. Therefore, you can use a different font for them

![Example of TilburgsAns](/images/2024-01-10-about-fonts/heading.png)

My love for [TilburgsAns](https://www.tilburgsans.nl/) is [no secret](https://jqno.nl/other/#tilburgsans). This font is an art project from my home town and it's used there for many different things, like event announcement posters, shop window signs, and wine bottle labels.

It's not a free font, but it has a very permissive license. Instead of paying for it, you can "[adopt](https://www.tilburgsans.nl/nl/over-ans/adoptieplan.html)" a letter (each letter can be adopted only once) or a space (unlimited supply). I have adopted space #256.

I've used it for various private projects over the years, including my daughter's birth announcement card. I've used it in the slides for [my talks](https://jqno.nl/talks/), and in fact, I'd already switched this site's headers to TilburgsAns a few months ago.

Butterick warns against using [goofy fonts](https://practicaltypography.com/goofy-fonts.html) in serious documents. I'm not sure if TilburgsAns qualifies as one, but I don't care, I'm keeping it.

## What I've learned

- Apparently, I prefer humanist fonts over (neo-)grotesque or mechanic ones. The [Monaspace website](https://monaspace.githubnext.com/) makes it easy to see the difference; generally, humanist fonts are more irregular. It's not a deal breaker for me though.
- Speaking of _grotesque_, in the context of fonts, it means something completely different.
- And speaking of words, when people say _font_, they really mean _typeface_, but no-one really cares and neither do I. I'll just continue saying _font_ even if I mean _typeface_.
- Font websites can be really _out there_. Check out [Gintronic](https://markfromberg.com/projects/gintronic), [Commit Mono](https://commitmono.com/), and [IBM Plex](https://www.ibm.com/plex/). The IBM Plex one is especially infuriating.
- Many fonts have configuration options, with descriptive names like `ss01`, `ss02`, and `cv01`, which define the shape of certain letters or ligatures. They're fun to play with!

## The end?

Of course, Butterick's [Practical Typography](https://practicaltypography.com/) has taught me a few things.

First of all, it brought home the point once more that making things available on the internet for free, is never actually really free. So if you've read his book as a result of this post, consider [paying for it](https://practicaltypography.com/how-to-pay-for-this-book.html). I have.

More importantly, the book has taught me that there's more to making websites look good than just fonts. Much, much more. Way more than I thought. And since reading it, there's loads of things I'd like to change about my website.

But I'm not going to do it. I'm not jumping into another rabbit hole.

At least not right now.

Therefore, I will leave you with a screenshot of what this page looks like _right now_. If you notice that this website looks very different, you will know that I've plunged into that rabbit hole after all. And that I've re-emerged. In that event, send me a message and ask me if I'm alright!

![Screenshot of what this site looks like in January 2024](/images/2024-01-10-about-fonts/screenshot.png)
