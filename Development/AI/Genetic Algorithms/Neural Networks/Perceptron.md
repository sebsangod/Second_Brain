---
aliases:
  - Perceptron
tags:
  - learning
  - dev/ai
date: 2026-05-10
---
**Sources**: [Introducción a la IA con Python](https://www.domestika.org/es/courses/5239-introduccion-a-la-ia-con-python)

**Related:** [[Artificial Intelligence]], [[Neural Network]], [[Activation Function]]

---

## Description

A structure that allows decisions to be made based on input variables:

![[perceptron.png]]

---

## Details

They have **Boolean inputs**, meaning their value can be 1 (if the condition is met) or 0 (if it is not met). **These inputs are referred to as** _X1_, _X2_, _X3_.

Each input can be **assigned a different** _weight_, that is, a _value_, denoted by _w_.

**They undergo processing**, which likewise returns a Boolean result: 1 or 0.

Within this processing, there is a variable called the _threshold_.

The processing determines whether **the sum of the inputs (X) is greater than the defined threshold**, resulting in 1 (it is greater) or 0 (it is not greater).


### Nowadays

Today, _perceptrons_ are referred to as _neurons_

Now, the threshold is called _bias_, and instead of performing a comparison, it is _added_.

The result is then passed to an [[Activation Function]], which finally returns the expected result:

![[neuron.png]]

---

## Claude Sessions
