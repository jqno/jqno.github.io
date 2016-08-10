---
title: "Foobal, Moneyball"
tags:
- foobal
- statistics
---
Ever since [Moneyball](http://www.imdb.com/title/tt1210166/) came out, winning at sports using statistics has become more and more mainstream. As I've [blogged before]({{ pctagurl('foobal') }}), I have also been getting in on some of that action: I've been competing in my family's soccer match betting game, where we guess the results of NAC Breda, our favourite team. But instead of predicting the outcomes myself, I'm using [Foobal](https://github.com/jqno/foobal), a little Scala program that I've written for this very purpose.

And, like the [Oakland A's](http://www.amazon.com/Moneyball-The-Winning-Unfair-Game/dp/0393324818), [FC Midtjylland](https://decorrespondent.nl/2607/How-data-not-humans-run-this-Danish-football-club/230219386155-d2948861) and [Brentford FC](http://www.theguardian.com/football/2015/jun/04/brentford-head-coach-marinus-dijkhuizen), this has not been without success: I won this year's pool! And by quite some margin, too. Here's the trophy:

![Trophy]({{ assets['Trophy'] }})

On the last few lines, it says `J10 anO15` which really should have been `2015 JanO`. I guess the thick black line got in the way :). I'll keep it until next year, when I pass it on to the next winner.

Anyway, Foobal probably did so well because NAC has been playing very, err... consistently this season. Sadly, due to this consistency, NAC were also relegated from the Honor Division to the First Division. I must say, it's sad to win the pool under such sad circumstances. To Foobal's moral credit, though, it did predict a win for the final and deciding match, which was lost during extra time.

The relegation also means that next year, NAC will have to face many teams that Foobal does not have data on. This will be quite a challenge, but it gives me a good opportunity to tweak the algorithm a bit and see if I can come up with something even more successful. I'm only getting started!
