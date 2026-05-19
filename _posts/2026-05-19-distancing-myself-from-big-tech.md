---
title: "Distancing myself from Big Tech"
excerpt: In which I replace services I've been using for literal decades
tags:
- big-tech
- google
- microsoft
---
The past few months I've been trying to distance myself from Big Tech. By that I mean the big American tech companies like Google, Microsoft and Meta, but also some of the smaller American companies like Dropbox. I want as much of my data as possible to be stored on servers in Europe, managed by European companies.

If you already feel the same way, feel free to skip to the [How](#how) section of this post. Otherwise, I invite you to just keep reading.

![A crude drawing of a figure walking away from the logos of some well known big tech companies](/images/2026-05-19-distancing-myself-from-big-tech/walking-away.webp)

## Why?

It's simple: I don't trust these companies anymore. That's because:

- they (well, especially Google) have a tendency to kill off services I use. It started in 2013 when Google killed [Google Reader](https://en.wikipedia.org/wiki/Google_Reader). Then in 2015, [Google Code](https://en.wikipedia.org/wiki/Google_Developers#Google_Code). And many more over the years. [The Good Cloud](https://thegood.cloud) are unlikely to ever kill their file storage product, since file storage is the only product they offer.
- they often don't provide support for their products. If something happens to your account (and there are plenty of stories of things happening to people's accounts), there's nowhere you can turn to to get it fixed. Even if they do provide support, there's often no way to contact a human when you need one.
- they keep stretching definitions and breaking promises, allowing themselves to reach further and further into my data so they can sell more and more of it to advertisers and who knows who else.
- they seem awfully friendly with Trump lately, doing almost everything he asks of them.
- they must follow [the CLOUD Act](https://en.wikipedia.org/wiki/CLOUD_Act), even if they do have good intentions (despite my previous point). This means that if the US government asks to look at my data, they are required by law to hand it over. Even if it's hosted in the EU. Europe trusts America not to actually do this, but that trust is based on [a gentleman's agreement made with a previous president](https://noyb.eu/en/us-cloud-soon-illegal-trump-punches-first-hole-eu-us-data-deal). We have since found out that not every US president is a gentleman. I may think I have nothing to hide, but Trump is quickly expanding on the list of things I may need to be careful about writing down and storing on an American server. I may have expressed political opinions in the past that were perfectly reasonable at the time, that some people, including Trump, now think is extremist. My opinions have not changed; the people in power have.

Also, Big Tech is no longer fun and exciting. I remember when smartphones were still fun and exciting. Cool, innovative new apps came out all the time. Over the years, Google has made it increasingly difficult for solo devs to publish indie apps to the Play Store, to the point where it's now almost a full-time job to even keep an app alive in the Play Store for a prolonged period of time. As a result apps show increasingly aggressive ads or require expensive monthly subscriptions. Or both. And I don't blame them. But it sucks.

Every time in the last ten years that I have been excited by some piece of tech, it was either an open-source project or an app maintained by a small company. Never by something produced by Big Tech. I miss that excitement, so I decided to start looking for it again.

<a id='how'/>
## How?

I've been looking for (and finding!) alternatives. My criteria are as follows, in descending order of desirability:

- open-source, run locally on my laptop or phone
- proprietary, run locally on my laptop or phone
- hosted service under European jurisdiction (either open-source or proprietary)
- self-hosted (I don't like maintenance but I'll do it if I have to)
- hosted service (data should be end-to-end encrypted)

There are several websites that I've used to find out about new services, such as the aptly named [European Alternatives](https://european-alternatives.eu) and [this forum post on Tweakers.net](https://gathering.tweakers.net/forum/list_messages/2285628/0).

While I made the conscious decision to distance myself from Big Tech at the beginning of this year (2026), I've actually been doing it for a longer time already, at a slower pace.

What I had already done before, in no particular order:

- **Social media**. I deleted my Facebook account many years ago. Shortly after the, erm, [incident](https://en.wikipedia.org/wiki/Elon_Musk_salute_controversy), I also deleted my Twitter account. These days I'm mostly on Mastodon, though I also still look at Bluesky from time to time.
- **Operating system**. I've been happily running Linux on my laptop for the last five years. I have found that Fedora strikes a really nice balance between stability and innovation.
- **Search**. I really like [Ecosia](https://www.ecosia.org), a German company. The quality of the results is good, and it uses its ad revenue to plant trees!
- **Analytics**. I removed Google Analytics from this site years ago, and replaced it with ... nothing. Really. I don't need the (lack of) validation it provides. Also, it's kind enough already that you choose to spend your time reading this; I don't want to punish that by having to shove a cookie notice in your face.
- **My phone, part 1**. I own a [Fairphone](https://www.fairphone.com), which I can repair without breaking warranty. I've had to replace several phones because of a malfunctioning USB port, but now I can just order a replacement for less than €10 and install it myself using only a screwdriver. I've already done it twice in the more than four years I've had the phone. It also has a removable battery. It's awesome.

What I've done this year (2026):

- **Thermostat**. This one was not intentional. In fact, accidentally spilling water on it, causing it to break during the coldest week of the year, leaving the house freezing cold, was not the smartest thing I ever did. Our plumber was kind enough to do an emergency repair by installing a new thermostat of a brand that he has a partnership with. It just so happened to be a European brand. Nice!
- **Domain registrar**. Even though the company I used to register my domain ([jqno.nl](https://jqno.nl)) was Canadian, not American, I think that my domain is so core to my online identity that it makes sense to keep it closer to home and move it to a Dutch one instead. Despite a few DNS hiccups, it was surprisingly easy to do.
- **Dropbox**. Technically not Big Tech maybe, but American nevertheless, and I used it for all of my notes as well as most of my photos. I settled on [The Good Cloud](https://thegood.cloud), a Dutch company that focuses on privacy. It uses Nextcloud under the hood, which integrates well with some other tools I was already using. I considered [Proton Drive](https://proton.me/drive) too, but decided against it because it's more locked into its own ecosystem, and my integrations would no longer work. (It felt weird shutting down my account, which was old enough to vote and drink in the Netherlands.)
- **My phone, part 2**. I went through all my apps and attempted to replace them with privacy-friendly alternatives. Sometimes, this was easy (Thunderbird for Android instead of the Gmail app), sometimes it was hard (keyboard app), sometimes it was impossible (WhatsApp). I've installed the [F-Droid](https://f-droid.org) app store, which provides all the cool open-source apps that are no longer on the official store. Unfortunately its future is kind of uncertain: Google wants to discourage sideloading (installing an app from something else than the official store), which makes it really hard for casual users to install and use F-Droid. (We need to [Keep Android Open](https://keepandroidopen.org/)!)
- **My phone, part 3**. After I replaced all my apps, I figured it's relatively pointless, since Google can see everything I do on my phone anyway. So I also installed a de-Googled OS: a variant of Android with all Google services and tracking removed. There are several options out there, but I went with [/e/OS](https://e.foundation/e-os), since that seems to be the one best supported for my Fairphone. Note that this isn't for the faint of heart: I nearly bricked my device in the process of installing it. Read the docs before you do this! Or: buy a (Fair)phone from [Murena](https://murena.com) with /e/OS pre-installed!
- **Google Wallet**. It has two features: being able to pay with my phone, and saving my loyalty cards. The loyalty card bit, I replaced with [Catima](https://catima.app), an open-source app that's both faster and easier to work with. I decided to stop using my phone to pay for things (in fact, I never really started doing that). Google really, _really_ doesn't need to be the middle-man for every single payment I make. I've always carried a small wallet in my pocket that holds 6 cards and not much else. It's perfect. I'll just keep doing that.

What I still have to do:

- **E-mail**. I already have a mailbox with Dutch e-mail provider [Soverin](https://soverin.nl) (through [Freedom Internet](https://freedom.nl)), but I need to use it more. I've used Gmail since it was still in beta and my friends and family still mail me there. Also, I still use my Gmail address to log into dozens and dozens of online accounts. I need to sit down and switch them over. A fun side-note is that I've connected my domain to my mailbox and set it up so that it doesn't matter what comes before the @ sign: it all goes to my inbox. This means that I have infinite e-mail addresses, and can use a different one for each service I sign up to. That way, if I receive spam, I will know where it came from.
- **Off-site backup**. I currently use Backblaze, which works well. I have set things up using [Restic](https://restic.net) to be end-to-end encrypted, so if the US authorities want to see my stuff, all they'll be able to get is encrypted data that neither they nor Backblaze themselves can decrypt. That's why it's not highest on my list of priorities. Nevertheless, I want to replace it. I haven't decided with what yet, though. I'll get to it at some point!
- **GitHub**. There have been quite a few people and projects leaving GitHub recently, and the community seems to have settled on [Codeberg](https://codeberg.org), which is German. I'll probably follow at some point, but haven't taken the time to do it yet. Apart from my personal projects, I'll need to migrate this website which is currently hosted on GitHub Pages, and [EqualsVerifier](https://github.com/jqno/equalsverifier) as well. I'll probably have to maintain a presence on GitHub for EqualsVerifier, because that's still where most people go to find open-source projects.
- **Todo app**. I'm something of a todo app power user, and I've found Todoist to be one of those rare apps that are both pleasant to use and have all the features that I want (why do so few todo apps properly support the concept of a start date?) Unfortunately it's American and doesn't do end-to-end encryption, so it'll have to go. Eventually. I'm dreading this one.

What perhaps can't be done:

- **Whatever Big Tech my employer uses**. Because I need to make a living, and I'm not going to convince a company with thousands of employees to ditch Sharepoint overnight.
- **Google Docs**. Outside of work, too, I sometimes need to collaborate with people on documents, which often happens with Google Docs.
- **Google Maps**. There are several alternatives out there, like [CoMaps](https://www.comaps.app), but none of them have real-time traffic information or reliable up-to-date opening hours for stores in my area. It's just too convenient.
- **YouTube**. I've been using [NewPipe](https://newpipe.net/) on my phone, which is nice (no ads!), but The Algorithm™ is just so good. It seems to know exactly the kinds of things I want to watch. I've been trying to replicate it by strategically subscribing to channels I like in NewPipe, but often there's great content from other creators that I simply would not know about without it.

## Conclusion

It's been a process, and it promises to remain a process for some time still. But I feel good about making this change. I know that me doing this will not change the world. I think the main reason many people stick with Big Tech is inertia: it's simply how they've been doing it all their digital lives, and like the proverbial boiling frog, haven't noticed how bad things have become. I hope this post may open some minds to the idea that change is actually possible, if not on a systemic level, at least for you as an individual.
