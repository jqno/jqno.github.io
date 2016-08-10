---
title: "Foobal: A New Hope"
tags:
- foobal
- bug
---
This weekend, a new soccer season started! It's way past time I told you all how I, or rather, how [Foobal](https://github.com/jqno/foobal) did last season.

As you may remember, Foobal started out [pretty well]({{ pcposturl(2012, 09, 11, 'foobal-predicting-soccer-matches-with-scala-and-drools') }}). After half a season, it was still doing [quite OK]({{ pcposturl(2012, 12, 31, 'foobal-end-of-year-update') }}). And then... not so much. Unfortunately. Foobal and I finished fourth; a position shared with three other people. In other words: slightly above the middle.

But I shan't be deterred! This year is going to be better. Actually, I know this for a SOLID FACT, because when I was dusting off the code and preparing the program for the new season, I discovered a bug. A bug that had been there the whole time. A bug that caused Foobal to ignore over 30% of all the data it could have used. A bug that caused foobal to ignore all matches that were played before the 10th of each month.

Oh dear.

How I haven't seen this sooner, I will never know. It's not like I didn't notice many matches were missing from the data file... But here we are.

It turns out that the website I'm scraping my data from, does something unexpected for single-digit dates: it pads them with an extra space. So, instead of `mar 2` or `mar 02`, it actually prints `mar  2` in the HTML. Note the extra space between `mar` and `2`.

The code that parses this date, uses a [regex](http://regex.info/blog/2006-09-15/247) that splits on spaces. So in the case of `mar 12`, it splits into `mar` and `12`, and then passes the 2nd part to the number parsing routine, which is fine. However, in the case of `mar  2`, it splits into `mar`, ` ` and `2`. And when it passes the 2nd part, which happens to be the empty string, into the number parsing routine, it throws an exception... which promptly gets eaten, never to be seen again. If ever you need an argument for a [fail-fast](http://en.wikipedia.org/wiki/Fail-fast) architecture, this is it.

To make matters even more amusing, the entire fix consisted of [one. single. character](https://github.com/jqno/foobal/commit/97e1852f2095bd14fa374288249d98cab19eaa7a). (OK, I updated some unit tests, too. But still.)

So! This year will be a lot better, I'm pretty sure. Even though Foobal's first prediction of the season was already incorrect. At least it's going to be based on much more accurate data.
