---
aliases:
  - Measures
tags:
  - learning
  - math/statistics
date: 2026-08-09
---
**Sources**: [Curso de Estadística y Probabilidad](https://platzi.com/cursos/estadistica-probabilidad/)

**Related:** [[Statistics]], [[Measures of Central Tendency]]

---

## Description

The ``mean``, _variance_, and _standard deviation_ are three statistical measures that allow you to describe a dataset using just a few values and, based on a sample, make inferences about an entire population.

---

## Key concepts

A _population_ is the entire group you want to analyze.

A _sample_ is a representative portion of that population.

A _parameter_ is a value that describes the entire _population_, such as the population mean (μ). It differs from a statistic, which describes only the sample (x̄).

---

## Details

### Mean

The mean is the **average**: the central value that distributes the data evenly. The formula is the same in both cases—the sum of the elements divided by the total—but the notation changes:

![[statistics_mean.png]]

- _Population mean_ $μ$: the sum of all elements divided by $N$ (total population size).
- _Sample mean_ $x̄$: the sum of all elements divided by $n$ (sample size).


### Variance

_Variance_ measures **how far each value is from the mean**. It is the sum of the squared differences between each data point and the mean, divided by the total number of data points:

![[statistics_variance.png]]


#### Why n − 1 and not n?

Because when you calculate a statistic from a sample, you introduce a bias. If you divided only by n, the variance would be underestimated. Dividing by n − 1 corrects that bias and gives you an unbiased variance. In other words, because estimating the variance from a sample introduces a downward bias. Subtracting one from the denominator compensates for that bias and yields a value closer to that of the population.


### Standard Deviation
The standard deviation measures **how far each value is from the mean**, but in the same units as your original data (not squared, like variance).

- If the standard deviation is large, the data is scattered: far from the mean.
- If it is small, the data is concentrated near the mean.
- If it is zero, all values are identical.

![[statistics_standard_deviation.png]]

---

## Claude Sessions
