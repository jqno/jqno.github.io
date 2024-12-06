---
title: "Un-refactoring"
excerpt: In which I throw away code that I'd already released.
tags:
- equalsverifier
- software-engineering
---
Yesterday was Sinterklaas, the Dutch holiday of gift-giving. Sinterklaas was very generous again this year for me and my family, but I also gave myself a gift: the gift of ... (checks notes) ... reverting two month's worth of work on EqualsVerifier!? That's, like, three whole releases!

So what happened?

In my [15 years retrospective]({% post_url 2024-06-01-looking-back-on-15-years-of-equalsverifier %}), I mentioned that if I could do it all over again, I would have taken a more immutable approach to the instantiation logic underpinning EqualsVerifier. That got me thinking: why don't I? I have more than a thousand unit tests on this sucker. I could [strangler fig](https://martinfowler.com/bliki/StranglerFigApplication.html) the heck out of that!

So I started. And it went well, until it didn't. Edge case after edge case popped up, causing exactly the kind of gnarly debugging sessions that I wanted to avoid by doing this. So I threw it away and started again, in a different way. And it went well, until it didn't. Different edge cases this time. And I got frustrated, because other cool projects that I also want to do got dismissed because of this.

I'd also been releasing various stages of this refactoring, in version 3.17 to 3.17.4, and issues started coming in for things that used to work well but now don't anymore. Corner cases that none of those 1000+ tests covered.

So, the other day I was doing a late-night debugging session prompted by a GitHub issue, tired, and starting to get migraine aura. Did you ever have migraine aura? It's like seeing a [Pink Floyd laser show](https://www.youtube.com/watch?v=-AC0ucyisd8) in your head, without the music and the ability to shut it off. Fortunately mine often come without the migraine headaches.

And that's when I decided: no more. The code works as it is. It has for the past 15 years. And generally without issues, too. Why would I pull the rug on that? I must be crazy. So yesterday, I pulled the trigger and reverted everything I made after the release of 3.17.1, because the first part of the refactoring didn't touch the nasty bits, didn't cause any of the issues, and was actually still valuable to me for my plans going forward.

I also went through all the commits that I reverted, and cherry-picked the things that were still good: mostly unit tests, but also fixes to a feature I'd introduced in 3.17 that didn't relate to the refactoring.

As we say in Dutch, 'beter ten halve gekeerd dan ten hele gedwaald': better to turn back halfway, than to go astray completely.

It feels painful in a way, but it's also liberating. Maybe [software companies](https://arstechnica.com/gadgets/2024/09/it-was-the-wrong-decision-employees-discuss-sonos-rushed-app-debacle/) should be brave enough to consider this option more often.
