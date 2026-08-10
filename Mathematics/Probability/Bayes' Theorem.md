---
aliases:
  - Bayes' Theorem
tags:
  - learning
  - math/probability
date: 2026-08-09
---
**Sources**: [Curso de Estadística y Probabilidad](https://platzi.com/cursos/estadistica-probabilidad/), [Better Explained](https://betterexplained.com/articles/an-intuitive-and-short-explanation-of-bayes-theorem/)

**Related:** [[Probability]], [[Machine Learning]], [[Data Analysis]]

---

## Description

_Conditional probability_ formalizes this idea: it calculates **the probability that one event will occur _given that_ another has already occurred**. When events are dependent, that “given that” changes the entire calculation.

This foundation prepares you to understand _Bayes’ theorem_, which delves deeper into **how to update probabilities when you receive new information**. It is one of the cornerstones of modern statistics and applications such as ``machine learning``, medical diagnosis, and spam filters.

---

## Key Concepts

### Conditional Probability

**When you flip a coin twice in a row, does the first result affect the second? And when you draw a card from a deck and don’t put it back, does anything change?** That difference is the basis of conditional probability, a key tool for understanding how one event influences (or doesn’t influence) the next and how to calculate their actual probabilities using the _multiplication rule_.

_Conditional probability_ **measures the likelihood of an event occurring when a previous event has already taken place**. In other words, it calculates how the outcome of a prior action may (or may not) alter the probability of the next step.

#### Multiplication rule

It states that the probability of two or more events occurring in sequence is obtained by multiplying the individual probabilities of each event.

##### Independent Events
_Independent events_ are those in which **the previous outcome does not affect the next one**. The classic example is flipping a coin.

If you flip a coin and get heads, that result does not influence the next flip. The probability of getting heads remains one-half, no matter how many times you flip it.

So, if you want to calculate the probability of getting heads and then heads again:

- Probability of heads on the first flip: $\frac{1}{2}$.
- Probability of heads on the second flip: $\frac{1}{2}$.
- Probability of heads followed by heads: $\frac{1}{2} * \frac{1}{2} = \frac{1}{4}$.

If you consider the _sample space_ for two coin tosses, you get four possible combinations: heads-heads, heads-tails, tails-heads, and tails-tails. Only one of them matters to you, so the probability is one in four. This matches the _multiplication rule_ perfectly.

###### Does the probability increase if you repeat the same event?

**No**. In fact, **it decreases**. And this is one of the most common mistakes people make when they bet on the same number over and over again, thinking they have a better chance.

Imagine a basketball player with a 70% chance of making each shot. What is the probability that he’ll make five shots in a row?

Since **each shot is independent**, you multiply: 0.70 × 0.70 × 0.70 × 0.70 × 0.70 = 0.168, or about 16.8%. You went from a 70% chance on a single shot to less than 17% on five. The probability drops quickly.


#### Dependent Events

_Dependent events_ are those in which **each action affects the next event**. The clearest example is a deck of cards.

A standard deck has 52 cards divided into four suits: hearts, clubs, diamonds, and spades. If you want to calculate the probability of drawing an ace, it’s simple: there are four aces among 52 cards, which equals $\frac{4}{52} = \frac{1}{13}$.

**But what if you then want to draw a king without returning the ace to the deck? That’s where the change comes in**. Now there aren’t 52 cards anymore—there are 51. So:

- Probability of drawing an ace first: $\frac{4}{52}$
- Probability of drawing a king afterward: $\frac{4}{51}$
- Combined probability: $\frac{4}{52} * \frac{4}{51} = \frac{16}{2652} \approx 0.6$%

**Every card you remove alters the sample space for the next attempt**. That is the essence of a _dependent event_: **the past does indeed condition the present**.

---

## Details

_Bayes' theorem_ allows us to calculate the **conditional probability of an event when you already know the individual probabilities and the inverse conditional probability**.

It's the tool that turns lengthy calculations into a straightforward formula—ideal if you're studying ``statistics``, ``machine learning``, or ``data analysis``.

$$
P(A|B) \ = \ \frac{P(B|A) * P(A)}{P(B)}
$$

---

## Claude Sessions
