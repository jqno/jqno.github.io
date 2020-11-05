---
title: "Holy serafins!"
tags:
- code
- literature
- references-in-popular-culture
- umberto-eco
excerpt: In which I discover a piece of BASIC code in Umberto Eco's novel Foucault's Pendulum.
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

