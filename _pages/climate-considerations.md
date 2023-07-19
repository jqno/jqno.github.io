---
permalink: /climate/
title: Climate considerations
layout: single
---

> TL;DR: the most effective way to help reduce CO<sub>2</sub> emissions as a software developer, is to reduce cloud spend.

Climate change is a big problem and I worry about it a lot. Maybe you do too. Perhaps you care about nature, or about living in a place that won't flood or burn, or where you won't melt during summers.

Maybe you care more about other things, like reducing migration: even then you have a stake in this, because people will migrate from places that become too hot. Or if you think dealing with climate change is too expensive or too ineffective: it will only become more expensive if we wait. But all small actions combined, do make a difference!

So what can we do?

As a single individual, it's hard to find a way to make a difference, but there are two things you can do: you could donate your money to organisations that deal with climate change, or you could donate your time by joining a political party (they're happy to have you!). And be mindful of what kind organisation you choose to work for, if you have the choice.

As a developer, depending on your position, you can actually do a lot. IT is responsible for a significant portion of CO<sub>2</sub> emissions, and there are things you can do to help reduce emissions for your organisation. The most effective thing you can do, is reduce cloud spend for your organization. The more you pay your cloud provider, the more machines you have running there. Do you need them all? Here's how you can reduce your spend:

- Turn off servers that you don't need. Do so aggressively.
- Cache your build artefacts. Downloading them again and again for each build makes your build slower anyway.
- Reduce uptime service level agreements (SLAs). 99.999% uptime guarantees require a lot of overhead in redundancy and orchestration. If you communicate to your users, they will be understanding of scheduled downtime.
- Look further than Spring Boot. Other frameworks can handle more requests per second, which means you'll need less machines to handle your load.
- Reduce network traffic and keep your html/JavaScript lightweight. Network traffic causes emissions too!

Other things you can look at, are: where is your data center located, and where does it get its power? Try to move your workloads to an appropriate location.

Want to know more? I recommend these resources:

- Free training: [Green Software for Practitioners at Linux Foundation](https://training.linuxfoundation.org/training/green-software-for-practitioners-lfc131/)
- Video: [Writing Greener Java Applications by Holly Cummins](https://www.youtube.com/watch?v=kwnnbvwXVXY)
- Book: [What We Owe The Future by William MacAskill](https://80000hours.org/what-we-owe-the-future/)
- Newsletter and other resources: [Green Software Foundation](https://greensoftware.foundation/)
