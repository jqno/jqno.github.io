---
title: "\"It's just test code\""
excerpt: In which I share how I managed to get my unit tests back under control.
tags:
- unit-testing
- java
---
"It's just test code" is something I hear way too often, from people who really should know better. Somehow, there is a pervasive sentiment that unit test code does not need to be held to the same standards as production code. I'm not sure why, but it seems to be associated with the idea that the "customer" will never "see" that code.

This kind of thinking is dangerous. As your code base grows, badly maintained test code will slow you down at least as much as badly maintained production code does.

I've seen tests that were so terse that I couldn't figure out what they were testing. I've seen tests that were so long and convoluted that I couldn't figure out how to fix them after they broke when I changed something in the production code. I've seen suites of tests that were so badly organized that I couldn't figure out where to find the tests for a specific feature. All of these were written by people who are paid to write this stuff. Some of them were even me!

For example, EqualsVerifier's test suite had devolved over the years into a state that it was virtually unmaintainable. Fixing bugs had become very hard and time-consuming. This was not because the production code was hard to understand; I'd kept that sufficiently well-factored. No, it was because it had become increasingly hard to fix the existing tests after a change, and to find a good location for a new test. When I did a training with [Michael Feathers](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052) last year, I asked him how I could best fix this. His advice was basically to start over from scratch.

I did not start over from scratch. I did, however, spend several weeks cleaning up and heavily refactoring the test code. Here are some things I've learned from that experience, and also from experience on other projects. I'll use a running example of classes modeling a card game.

## Test naming
This is the most important thing. You might understand what a test is supposed to test when you're writing it, but will you still understand it in three months? Will somebody else?

For example, one of your test methods might be named `testShuffle()`. First of all, we're not living in the nineties anymore. With JUnit 4's `@Test`, there's no need to prefix all your test methods with the word test. It's just clutter, so get rid of it. Second of all, I can see that this test probably performs a test on the `shuffle()` method. But what does it test, exactly? The happy path? If so, which happy path? Maybe it even tests all possible interactions with this method? If so, it'll be quite a long test method, I suppose.

Another example of a badly named test is `shuffleFails()`. Why does it fail? And why is it supposed to? How am I supposed to know? I could look at the code, but even that might not tell me everything I need to know. It might be specified behavior, or an observation of a known bug that I want to change later.

For EqualsVerifier, I've started using a modified "[given-when-then](http://martinfowler.com/bliki/GivenWhenThen.html)" style for naming my test methods. I reverse the order to "then-when-given", and I often leave out the "given" or "when" part when I don't need them. I like to start the name with the method I'm exercising (if appropriate), and I separate the parts with an underscore for extra readability. For example: `shuffleReshufflesTheDeck_givenAPreShuffledDeck()`, `dealGivesThreeCards_whenRequestingThreeCards_givenAShuffledDeck()`, `dealThrowsIllegalStateException_whenCalled_givenAnUnshuffledDeck()`.

Yes, those are long Java names, but it beats having a shorter name and putting the "given-when-then" description in a comment for several reasons. First of all, you won't have to invent a short name for your test in addition to the long "given-when-then" name. [Naming things is hard](http://imgur.com/y3dhJFQ), so this is one less thing you need to name. Secondly, the method name is picked up by your test runner, while any comments are not. So if a test fails, you immediately know what's up; you don't need to go to the test to read the "given-when-then".

In languages and/or framworks that support it, I always prefer a string over an identifier to name my tests. For example, in ScalaTest you can also write:

{% highlight scala %}
"shuffle" should "reshuffle the deck given a pre-shuffled deck" in {
  // ...
}
{% endhighlight %}

Obviously, this is much more readable than a camel-cased Java identifier.

## Magic numbers
Many people believe magic numbers are OK in unit tests. I think, however, that introducing constants can be very beneficial in making a test more understandable. For example:

{% highlight java %}
@Rule ExpectedException thrown = ExpectedException.none();

@Test
public void dealThrowsISE_whenAsking79Cards() {
    thrown.expect(IllegalStateException.class);
    deck.deal(79);
}

@Test
public void dealThrowsISE_whenAskingTooManyCards() {
    thrown.expect(IllegalStateException.class);
    deck.deal(NUMBER_OF_CARDS_IN_A_DECK + 1);
}
{% endhighlight %}

Not everybody might immediately recognize that `79` is not some aribitrarily large number, but that it is, in fact, precisely one more than the number of cards in the [French game of Tarot](http://en.wikipedia.org/wiki/French_tarot). In the second example, it's immediately clear what's going on. You don't even need to see the actual value for `NUMBER_OF_CARDS_IN_A_DECK`.

Here's another example:

{% highlight java %}
@Test
public void dealGivesTheExactNumberOfCardsAsked_whenAskingSomeNumber() {
    List<Card> hand = deck.deal(SOME_NUMBER);
    assertThat(hand.size(), is(SOME_NUMBER));
}

@Test
public void dealGivesAFullHandOfCards_whenAskingTheNumberOfCardsInAHand() {
    List<Card> hand = deck.deal(NUMBER_OF_CARDS_IN_A_HAND);
    assertThat(hand.size(), is(NUMBER_OF_CARDS_IN_A_HAND));
}
{% endhighlight %}

The difference may be subtle, but in the first case, I don't care about the specific number as long as it's the same in both instances, whereas in the second one, I do care about the specific number. The first test looks like it's part of a test suite for a generalized `CardDeck` class, while the second one might be specific to a particular game.

Note that magic numbers aren't necessarily numbers; they can be `Strings` or even entire objects too. When I don't care about the specific magic number, I like to prefix it with the word "Some".

## Refactor
As stated above, I've seen single test methods that were over twenty lines in length. They often contain comments which may or may not have gotten stale, lots of initialization code that may or may not be copy-pasted over several different test methods in slightly modified forms, and lots and lots of assertions checking tiny little fields on a big object. When one of these tests fails, or you've made a change in the production code that requires the test to change as well, it might take a very long time to figure out what's going on in that test, what it's actually doing, and what it should be doing. Also, if one assert fails, you don't know in advance how many of the following asserts in the method will fail as well. You don't have all the information that you should be having.

In other words, these tests tend to be very brittle, leading to lots of frustrating time wasted in maintaining them. So how to fix this smell?

It should be obvious, really. Test code is code, so you can refactor it. Move initialization code to an `@Before` set-up method, or to a private helper method. Give the helper method parameters if you have different flavors of initialization. Big assert blocks can be moved to a helper too. So instead of this:

{% highlight java %}
    assertThat(hand.get(0).getValue(), is(Value.ACE));
    assertThat(hand.get(0).getSuit(), is(Suite.SPADES));
{% endhighlight %}

You should have something like this:

{% highlight java %}
    assertCard(hand.get(0), Value.ACE, Suite.SPADES);

// ...

private void assertCard(Card actualCard, Value expectedValue, Suite expectedSuite) {
    assertThat(actualHand.getValue(), is(expectedValue));
    assertThat(actualHand.getSuit(), is(expectedSuite));
}
{% endhighlight %}

However, as I said, you shouldn't be asserting too much in a single test, because you'll lose information. If the first assert fails, you no longer know what the rest will do. It might be the difference between a typo and a major design flaw and you won't know until you've fixed the first assertion and the second one can be executed, and the third, etc. But when you refactor this test into several smaller ones, you'll immediately see how big your problems are just by the number of tests that are failing.

By refactoring mercilessly, your test methods will become shorter and clearer. It will become easier to understand what a test does, and it will become easier to maintain a whole suite of tests, because if something changes in the initialization, there's now only one place where you need to make that change, instead of lots of different individual test methods.

Additionally, remember those comments that may or may not have gotten stale? If you've named your helper methods well, you probably don't need those comments anymore. Just sayin'.

This is what I mean when I say that test code is code, too. You don't tolerate excessive copy-pasting in production code; why would you tolerate it in test code?

## Arrange, act, assert
This is the same thing as "given-when-then", and it's a good way to organize your test method. Consider:

{% highlight java %}
@Test
public void dealGivesKnownCards_whenShufflingWithAKnownSeed() {
    Random rand = new Random(TEST_SEED);
    CardDeck deck = new CardDeck(rand);
    assertThat(deck.size(), is(52));
    deck.shuffle();
    List<Card> hand = deck.deal(2);
    Card one = hand.get(0);
    assertCard(one, Value.ACE, Suite.SPADES);
    Card two = hand.get(1);
    assertCard(two, Value.QUEEN, Suite.HEARTS);
}
{% endhighlight %}

versus:

{% highlight java %}
@Test
public void dealGivesKnownCards_whenShufflingWithAKnownSeed() {
    // Arrange
    Random rand = new Random(TEST_SEED);
    CardDeck deck = new CardDeck(rand);
    deck.shuffle();

    // Act
    List<Card> hand = deck.deal(2);
    Card one = hand.get(0);
    Card two = hand.get(1);

    // Assert
    assertThat(deck.size(), is(52));
    assertCard(one, Value.ACE, Suite.SPADES);
    assertCard(two, Value.QUEEN, Suite.HEARTS);
}
{% endhighlight %}

This is obviously a contrived example, but in general, it's nice to always keep tests in the same order, so you will always know where to find things in tests that you may not have seen in a while. Note that you don't always need an Arrange section, as it might be covered by the `@Before` set-up method. If I'm unable to disentangle your test in this way, I see it as a signal that I need to refactor the test into several smaller ones. If any of the blocks becomes more than a few lines in length, I will start extracting methods.

By the way, I don't actually write the comments in real life, I added them here for educational purposes.

## Static analysis
If you use static analysis tools like FindBugs or CheckStyle, do you apply them to your test code as well? Because you should. Use the same configuration that you use for your production code, but with a few exceptions. For example, in test code, it's ok if test methods throw `Exception`, and test classes can have non-static public fields (as they are required for JUnit's Rules). Therefore, you can disable the corresponding CheckStyle checks for your test code.

## Conclusion
Test code is not "just" test code; you should treat it as if it were production code, and keep it to the same standards. Refactor it. Take care of it. Love it. If you do, you will end up with a test suite that you can actually maintain. Future-you will love you for it.
