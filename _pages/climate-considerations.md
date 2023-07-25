---
permalink: /climate/
title: Climate considerations of a software developer
layout: single
---

> TL;DR: the most effective way to help reduce CO<sub>2</sub> emissions as a software developer, is to reduce cloud spend.

Climate change is a big problem and I worry about it a lot. Maybe you do too. We've all seen the floods, the forest fires and the heat waves on the news.

Climate change is traditionally a topic of the political left, but I don't understand why. The political right should care about it too: climate change will have severe and unpredictable consequences that should worry conservatives who want to preserve their way of life; it will cause large numbers of people to attempt to migrate from more affected places in Africa and Central America to less affected places in Europe and the USA; and yes, dealing with climate change is expensive, but not dealing with climate change is even more expensive. Besides, green tech simply is good business these days.

So what can we do?

As a single individual, it's hard to find a way to make a difference, but there are two things you can do: you could donate your money to organisations that deal with climate change, or you could donate your time by joining a political party (they're happy to have you!). And be mindful of what kind organisation you choose to work for, if you have the choice. If you work for Big Oil, you might be in the wrong place.

As a developer, depending on your position, you can actually do a lot. IT is responsible for a significant portion of worldwide CO<sub>2</sub> emissions, and you can help reduce emissions for your organisation. The most effective way is to reduce cloud spend for your organization. The more you pay your cloud provider, the more machines you  probably have running there. Do you need them all? Here's how you can reduce your spend:

- Turn off services that you don't need. Do so aggressively.
- Cache your build artefacts. Downloading them again and again for each build makes your build slower anyway.
- Reduce uptime service level agreements (SLAs). 99.999% uptime guarantees require a lot of overhead in redundancy and orchestration. If you communicate well to your users, they won't mind the occasional scheduled downtime.
- Look further than Spring Boot. Other frameworks can handle more requests per second, which means you'll need less machines to handle the same load.
- Reduce network traffic and keep your html/JavaScript lightweight. Network traffic causes emissions too!

Other things you can look at, are: where is your data center located, and where does it get its power? Try to move your workloads to an appropriate location where there's a lot of renewable energy.

Want to know more? I recommend these resources:

- Free training: [Green Software for Practitioners at Linux Foundation](https://training.linuxfoundation.org/training/green-software-for-practitioners-lfc131/)
- Video: [Writing Greener Java Applications by Holly Cummins](https://www.youtube.com/watch?v=kwnnbvwXVXY)
- Book: [What We Owe The Future by William MacAskill](https://80000hours.org/what-we-owe-the-future/)
- Newsletter and other resources: [Green Software Foundation](https://greensoftware.foundation/)
