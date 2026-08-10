---
aliases:
  - Probability
tags:
  - learning
  - math/probability
date: 2026-08-09
---
**Sources**: [Curso de Estadística y Probabilidad](https://platzi.com/cursos/estadistica-probabilidad/)

**Related:** [[Mathematics]], [[Statistics]]

---

## Description

_Probability_ **measures how likely an event is to occur**, and it becomes essential when you work with ``statistics`` applied to real-world experiments.

In simple terms, _probability_ is the **science that measures the certainty that an event will or will not occur**. Its value always lies between 0 and 1, or between 0% and 100%.


### Why _Probability_ Complements ``Statistics``

``Statistics`` **collects and analyzes data**, but that data usually comes from experiments designed to resemble the real world. That’s where _probability_ comes in: it **tells you how reliable your results are**.

---

## Key concepts

The _event_ is what you want to calculate the probability of — that is, the outcome that meets your criteria.

The _sample space_ is the complete set of possible outcomes in the experiment.

Think of a six-sided die. Your sample space ranges from one to six because there are six possible outcomes. If your event is rolling a two, only one face meets that criterion. That’s why the probability of rolling a two is 1 in 6.

---

## Details

### Simple Probability
$$
P(Event \ A) = \frac{Events \ that \ meet \ the \ specified \ criteria}{Total \ sample \ space}
$$
The key condition is that the _sample space_ be uniform: **each outcome must have an equal chance of occurring**, like the dice example with probabilities of $\frac{1}{6}$.


### Addition Rule, Union and Intersection

Now, let’s consider calculating the probability of obtaining a specific outcome when rolling the dice. For example, the probability that at least one of the dice shows a ‘1’.
By examining our sample space, we determine that there are 11 possible combinations in which at least one ‘1’ appears. This translates to a probability of $\frac{11}{36}$.

Expanding on this example, we could also calculate the probability of rolling an even sum when rolling both dice. Since there are 18 possible combinations that add up to an even number, the probability is $\frac{18}{36}$.

#### How is the addition rule used?

At this point, you might think that adding the two previous probabilities is enough to find the probability of rolling a ‘1’ or an even sum. However, this is where the _intersection_ comes into play: **some outcomes satisfy both conditions** (such as rolling two '1's).

The formula for the addition rule tells us:
$$
P(A \cup B) = P(A) + P(B) - P(A \cap B)
$$

Applying this, we subtract the 5 events that are counted twice (their intersection on a Venn diagram), correctly adjusting the probability to $\frac{24}{36} = \frac{2}{3}$.

#### What about independent events?

What if we want to calculate the probability that the sum of the dice is even or odd? In this case, the sum cannot be both even and odd at the same time. **If two events are mutually exclusive** (as in this case), **their** _intersection_ **is zero**. Therefore, the probability of getting an even or odd sum is simply the sum of their individual probabilities.

---

## Claude Sessions
