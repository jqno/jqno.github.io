---
title: "Foobal: predicting soccer matches with Scala and Drools"
tags:
- foobal
- scala
- drools
- soccer
---
Introduction
------------

In my family, there is a long-standing game of betting on the outcome of "our" favourite soccer team: NAC Breda. The rules of the game are simple: before each match, you must predict the outcome of the match, and text the prediction to my uncle. If you were right, you get two points. If you were wrong, but predicted correctly who won or lost (or that it was a draw), you get one. Otherwise, you get zero. Whoever has the most points at the end of the soccer season, wins.

A few months ago, I realised that I was just about the only one who doesn't play. Now, I like games just as much as the next guy; I just don't like soccer that much. But I do like the betting game, especially since no real money is involved: that makes it safe to lose :).

So I decided to join the game. There were only two problems: first, I know next to nothing about soccer, and second, I'm quite comfortable with not knowing much about soccer and I'm not averse to keeping things that way. Fortunately, being a programmer, I had a simple solution for both problems: I would simply write a program to predict the outcomes! And I did. I'd like to tell you something about it.


Foobal
------
First of all, a program should have a name. I decided to name it _Foobal_, as a reference to a well-known [metasyntactic variable](http://en.wikipedia.org/wiki/Foobar) and, of course, because it kind of sounds like "football" (or "voetbal", if you prefer). Pun = fun.


How it works
------------
The idea behind Foobal is to gather data of previous matches, and use this data to make an informed prediction of the next match. Gathering the data is easy; there are many websites that aggregate the Dutch soccer match results. A simple scraper suffices. The scraper gathers the data and writes it to an XML file, which serves as input to the prediction engine.

This part I wrote in Scala. I used its native XML support to scrape the HTML (because everybody knows you [shouldn't use regex](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454#1732454)), and do I/O on the XML data file. Of course the website that I scraped did not consist of properly formatted, validated XHTML, and Scala's default XML parser failed miserably trying to make sense of it. Fortunately, it turned out to be pretty easy to hook [TagSoup](http://ccil.org/~cowan/XML/tagsoup/) into Scala's native XML support, like this:

{% highlight scala %}
private def parseHtml(input: String): scala.xml.Elem = {
  import org.ccil.cowan.tagsoup.jaxp.SAXFactoryImpl
  val parser = new SAXFactoryImpl().newSAXParser
  XML.withSAXParser(parser).loadString(input)
}
{% endhighlight %}

Next, I created a simple case class, called `Outcome` for lack of a better name, to store the results of my scraping. It consists of the names of the home team and the out team, their respective scores, and the date of their match. This case class can easily be written to and read from XML, without duplicates.

Making predictions is the interesting part. At a previous job, I learned to work with the [Jess](http://www.jessrules.com/) rule engine, which seemed like a good fit for this problem. Rule engines are very good at extracting information from a large amount of data and making decisions based on this information. They're fast, and the code is a lot more readable than layers and layers of nested if/else statements; not by a small margin, either. It took me some time to wrap my head around the concept, but when I finally did, it was extremely powerful. Unfortunately, Jess is not free, so I went with [Drools](http://www.jboss.org/drools/) instead. Its syntax is quite different from Jess, but the idea is the same. First, you define some rules and you insert some data (in this case, a bunch of `Outcome` objects). Then you let the engine run. Using [an interesting algorithm](http://en.wikipedia.org/wiki/Rete_algorithm) which I won't try to explain here, the engine lets the rules interact with the data. Once no more interactions are possible, the engine is done and you can collect the results.

So I built a little framework that can execute multiple prediction strategies simultaneously, and aggregate the results. For example, one strategy could be "just pick a random number". Another could be "for both teams, take the average of the last six matches rounded up". Each of these strategies is encoded in a single Drools file. The framework collects the results from each strategy, and takes the median. So, say I have three strategies, and they return the outcomes 2-2, 0-0 and 0-1. In other words, the possible scores for the home team are 0, 0, 2; the scores for the out team are 0, 1, 2. Take the median of each, and combined, the final outcome would be 0-1.

This approach has an interesting property, my uncle told me when I explained the program to him. It calculates a score for the home team and one for the out team, and puts these together. Only then can you see which team wins or loses (or if there is a draw). Whereas traditionally, when predicting a score for a soccer pool like this, one would decide first which team wins and which one loses (or if there is a draw), and only then choose a score based on this decision. In other words: Foobal does things the 'wrong' way round. I like to think that my lack of knowledge in the field allows me to think outside the box :).


Results (so far)
----------------
So, of course, after having read this wall of text, you'll want to know how Foobal is doing. Well, after four matches, it has gathered 6 points, out of a possible 8. Not too shabby!

It's way too early to draw any conclusions yet, but my uncle told me he's kinda getting worried I might actually win. We'll see; there are still 34 more matches to go, after all. Anyway, win or lose, I will keep you posted.


Wishlist
--------
So, Foobal is actually doing quite well. Much better than I expected, anyway. So for now, I'll leave the project as is and focus on other projects. But I definitely have some ideas for future improvements:  

* First of all, I'd like to re-implement my "average of last 6 matches" strategy, because its current implementation is quite messy. However, I don't know enough about Drools yet to know how. I've put the problem up on [StackOverflow](http://stackoverflow.com/q/12090295/127863), but so far I haven't had much luck.
* I should implement more strategies. So far, I've implemented only two: "take the average score of the last six matches", and "take the score of the same match last year". The first one helps keeping track of how well NAC Breda and its opponent have been doing recently, wherease the latter helps mitigate the problem that NAC has a tendency to play well against better teams, and badly against worse teams. But still, many more strategies are possible and it feels like a waste of a perfectly good rule engine framework to only implement two strategies on it :).
* I want to have a better way of aggregating the results of the various strategies. Initially, I thought of taking the mean, but I was afraid that it would be too "difficult" to predict a zero outcome (due to rounding). So I switched to the median, which (probably) doesn't have that issue. However, since I have only implemented two strategies yet, [by definition](http://en.wikipedia.org/wiki/Median) I'm taking the mean anyway. Oh well. In either case, scores will probably 'even out', so Foobal will probably never predict something extreme, which is sad, because sometimes an extreme prediction can be plausible, too.
* I want to measure the performance of the individual strategies, so that I know which ones work well, and which ones don't. Because basically, I'm flying blind at the moment. I could use this data to come up with some sort of weighted average, which would help me tremendously with the previous wish. Also, I'd like to automate these measurements, and I should probably also continuously keep doing them to adjust the weights as the season progresses.
* I'd like to have a nicer user experience, even though I'm the only user. At present, Foobal only has a command-line interface, requiring me to type out entire team names. That's kind of a pain, with names like "Roda JC Kerkrade" and "SC Heerenveen". The fact that everything is case-sensitive doesn't help, either. So I would like to have something to help me with that. If it can scrape previous outcomes, surely I can make it scrape the opponent for the next match somewhere. Also, while I'm at it, I'd like to turn it into a web app so I can run it from anywhere I want, and maybe it could even send an SMS message to my uncle automatically.
* Finally, the XML output currently isn't very friendly on the human eye. Foobal just poops the data on a single line with no formatting whatsoever. In theory, this doesn't matter, but for debugging it's sometimes nice to be able to load the data into an editor and look at it.

Long story short: there's still lots of work ahead!


Conclusion
----------
So, there you have it, folks: a soccer match predicter, written in Scala and Drools! I had a lot of fun writing it, and I had many interesting lunch-time conversations about it with my co-workers. Also, I learned a lot. Not about soccer, obviously; that was the point. But about Scala. For example, I gained some experience doing TDD in Scala, and I discovered a nice way to do dependency injection, which I may write about in a future post.

By the way, Foobal is open source, so if you want to join a soccer pool yourself, you can [fork it on GitHub](https://github.com/jqno/foobal). If you do, please let me know what you think of it!
