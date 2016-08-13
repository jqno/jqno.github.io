---
title: "Holy serafins!"
tags:
- code
- literature
- references-in-popular-culture
- umberto-eco
---
Who would expect, while reading High Literature, to see a piece of BASIC code
actually included in the text? Not me, that's for sure. But if you read Umerto
Eco's novel Foucault's Pendulum, it can happen to you too!

For those of you not familiar with this excellent novel: Foucault's Pendulum is
basically a thinking man's Da Vinci Code. One that doesn't suck. At all. Or, as
[Eco himself put it](http://www.theparisreview.org/interviews/5856/the-art-of-fiction-no-197-pauleacute-baacutertoacuten):

> The author, Dan Brown, is a character from Foucault's Pendulum! I
> invented him. He shares my characters' fascinations--the world
> conspiracy of Rosicrucians, Masons, and Jesuits. The role of the Knights
> Templar. The hermetic secret. The principle that everything is connected. I
> suspect Dan Brown might not even exist.

But I digress.

The main characters are trying to uncover some ancient secret by generating all
permutations of God's name: JHVH (of which, incidentally, there are 24). Jacopo
Belbo has just bought a computer which he loves as if it was his first-born
child, even though it's not an Apple product. He actually gives it a name:
Abulafia. So, obviously, Abulafia is called upon to generate the permutations,
using the following program:

{% highlight vb %}
10 REM anagrams
20 INPUT L$(1),L$(2),L$(3),L$(4)
30 PRINT
40 FOR I1=1 TO 4
50 FOR I2=1 TO 4
60 IF I2=I1 THEN 130
70 FOR I3=1 TO 4
80 IF I3=I1 THEN 120
90 IF I3=I2 THEN 120
100 LET I4=10-(I1+I2+I3)
110 LPRINT L$(I1);L$(I2);L$(I3);L$(I4)
120 NEXT I3
130 NEXT I2
140 NEXT I1
150 END
{% endhighlight %}

Quite the clever algorithm, I might add, although Eco doesn't address the
duplicates this program will produce when given God's name as input.

Has anybody else ever encountered real, actual code in a novel/movie/billboard?


Comments from my previous blog
------------------------------

#### Ralf responded:

> Actually, Eco does address the duplicates. In the words of Diotallevi: "(...) l'arte della Temurah non ti dice che devi permutare le ventisette lettere dell'alfabeto ma tutti i segni della Torah, dove ogni segno vale come se fosse una lettera a se stante, anche se appare infinite altre volte in altre pagine, come a dire che le due he del nome di Ihvh valgono come due lettere diverse."
>
> Thanks to your post I found another (albeit quite superficial) link in the novel. When Abulafia has spit out an alternate history at the end of chapter 65, and Casaubon remarks that this version corresponds to an existing (man-made) theory, Diotallevi uses the exact same phrase he uses in chapter 5 ("Santi Serafini") when he learns that Abulafia is re-inventing Temurah by means of the permutation algorithm, and now adds a nicely appropriate "Lo avevo detto." :-)

#### [Jan Ouwens](http://www.jqno.nl) responded:

> I'm still trying to think of a witty response to this, but my Italian isn't so good. So I guess that, as usual, I'll have to defer to your superior knowledge of this novel :).<br>
> Glad that I was able to give you a new insight though, however small it might have been!

#### Snader responded:

> In Transformers the Movie from 1986 they show BASIC code. Not quite sure if it would run, since you only get to see a fraction of a huge complex AI :P

#### [Jan Ouwens](http://www.jqno.nl) responded:

> Hm, I'll have to watch it again, then :).
>
> Also, implementing an AI in BASIC? That's ... that's awesome.
