---
title: "Foobal: end-of-year, half-of-season update"
excerpt: In which I manage to delight and annoy my uncle even more.
tags:
- foobal
- statistics
- soccer
---
Introduction
------------

In [a previous post]({% post_url 2012-09-11-foobal-predicting-soccer-matches-with-scala-and-drools %}), I promised to keep you updated on the progress of [Foobal](https://github.com/jqno/foobal).

After about five or six weeks, Foobal was squarely in the lead of the family pool. However, after that, Foobal got more and more predictions wrong. It seems to be slightly biased towards tie games (a score of 1-1 was a particularly common prediction), which aren't all that common in practice. So I decided to fix that. And of course, the two matches after that ended in a tie that Foobal sadly did not predict. Bummer. One of the matches would even have been predicted correctly had I not changed anything. Double bummer. After that, however, Foobal started to get some predictions right again. I'm confident that my changes will work out for the better. I won't be deterred by statistical anomalies!

Currently, we're halfway through the season, and the soccer competition is on winter hiatus. Every team has played every other team exactly once, and the second round will start at the end of January. Foobal currently holds a (shared) third place in the family pool, with 11 points out of a theoretically possible 38. The number one already has 18 points, wich will be very tough to beat!


Recap
-----

Before I explain what I changed, I will give a quick recap of what was already there. Foobal allows for several 'predicters' to make a prediction. It will then take all these predictions, and take the median of the scores for each team. Note that the resulting prediction may (and probably will) be different from all of the individual predictions. Note, also, that taking the median implies making a 'conservative' prediction: the extreme scores simply don't lie in the middle.

So far, I had implemented two rules. The first one simply 'predicts' the outcome of last year's match, or 0-0 if the teams didn't play each other last year. The second rule takes, for both teams, the average of the total number of goals scored in the preceding six matches.


The 'pessimism factor'
----------------------

I noticed that Foobal rarely predicted a zero score, even though it's quite common in practice for one, or even both, teams to score no points in a given match. This was due to two facts: first, I only had two rules. Taking the [median](http://en.wikipedia.org/wiki/Median) of two values is, by definition, taking the average of these values. So for Foobal to predict a zero score, both predictions had to be 0, or otherwise the average would be 0.5, which gets rounded up to 1.

Secondly, the 'average of the last 6 matches' rule also takes an average (obviously). For this rule to predict a zero, the average would have to be less than 0.5, or else it, too, would get rounded up to 1. The average would therefore have to be between 0.0 and 0.5 to return 0, and between 0.5 and 1.5 to return 1. That's a 'range' of 0.5 points to get 0, and a full 'range' of 1.0 to get 1 (or higher). In other words, the chance of this rule predicting 0 is much smaller than for it to predict 1, making it all the more unlikely that the average of both rules would yield a 0, too.

To fix this issue, I introduced a 'pessimism factor'. The rule now subtracts 0.5 from any given average, evening out the ranges. The chance of a 0 prediction is now equal to the chance of a 1 prediction. I realise that mathematicians might object to this kind of reasoning, but I'm not a mathematician, and this seems to work well enough :).


The leaderboard
---------------

The second way to make it more probable to predict a zero, is to add a third rule. That way, Foobal won't have to take an average anymore, eliminating the rounding issues. Of course, taking the median means that two of the rules will have to predict a 0 for Foobal to predict zero as well, so it would be nice if this new rule occasionally predicts a zero, too. But what rule to add?

I decided to implement a rule that uses the soccer competition's [leaderboard](http://teletekst.nos.nl/?819-01). It looks up both teams, and gives the weaker team an automatic score of 0. The stronger team gets a score equal to half the distance between the two teams. If the teams are close, this results in a score of 0-0 or 1-0, which seems reasonable. If the teams are far apart, the rule might predict scores of up to 9-0. This is quite a big score, but since Foobal takes the median of all predictions, this score will never be the final prediction (unless one of the other rules predicts it, too). What it does, however, is nudge the final prediction towards the bigger of the predictions of the two other rules. This feels reasonable: this rule reflects the fact that a team will, generally, perform better against a weaker team, than against a weaker one.

Of course this meant that I had to implement a leaderboard, too. I could scrape this off a website the same way I scrape the outcomes of the individual matches, but I prefer to keep the scraping to a minimum, as it breaks easily. Besides, deriving a leaderboard from the individual outcomes seemed like a fun programming exercise! The leaderboard that I ended up with isn't perfect, however: it ignores the number of goals scored when two teams are tied for the same position on the leaderboard. In such cases, the 'winner' is currently undecided. This means that Foobal's leaderboard isn't very accurate, as such ties are quite common in practice. At the moment of writing, there are five such ties on the official leaderboard, involving more than half of all the teams. However, I decided not to care: in practice, the strength of these tied teams won't differ a lot, and it adds a small, controlled amount randomness to the whole thing, which I kind of like.


The home team advantage
-----------------------

Another idea that I had, but did not implement, was to give the home team an advantage over the visiting team. As I mentioned above, with two rules, Foobal would take the average of both rules's predictions. By rounding the home team's score up, and the visiting team's score down, the home team would gain an advantage. However, somehow I felt this advantage would be too big. And anyway, the point became moot when I added the third rule. Maybe I'll reconsider it if I decide to add a fourth rule, though.


Conclusion
----------

So that's what I did to improve Foobal's predictions. Did I measure any of my claims before implementing them? Nope. Did I measure any of them after implementing them? Again: nope. This is something that I really should do, but haven't gotten around to yet. Maybe when we start betting for money. Until then, the purpose of this project is merely to have fun, to learn something in the process, and maybe to scare the occasional uncle.

