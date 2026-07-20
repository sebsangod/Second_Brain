---
aliases:
  - Derivate
tags:
  - learning
  - math/differential-calculus
date: 2026-05-09
---
**Sources**: [Curso Básico de Cálculo Diferencial para Data Science e Inteligencia Artificial](https://platzi.com/cursos/calculo-diferencial-ds/), [Statistics How To](https://www.statisticshowto.com/derivatives/#definition)

**Related:** [[Differential Calculus]], [[Limits]], [[Function]], [[Neural Network]], [[Backpropagation]], [[Gradient]]

---

$$f’(x) = \frac{dy}{dx} = \lim_{h \ \to \ 0} \frac{f(x+h)-f(x)}{h}$$

___

## Description

Simply put, **it’s the instantaneous rate of change**. It tells you how quickly the relationship between your input (x) and output (y) is changing at any exact point in time.

![[derivate_graph.png]]

___

## Chain Rule

The _chain rule_ **is used to differentiate composite functions**. Remember that a composite ``function`` is a ``function`` that takes another ``function`` as an argument.

Using the _chain rule_, **we can find the rate of change of an initial variable with respect to a final variable**. This is used in ``neural network algorithms`` such as ``backpropagation``.

To do this, we multiply the _derivative_ of the outer ``function`` by the _derivative_ of the inner ``function``:

$$ y \ = \ f(g(t))$$
$$ y´(f(g(t))) \ = \ f´(g(t)) \ * \ g´(t) $$
$$\frac{dy}{dt} \ = \ \frac{dy}{dg} * \frac{dg}{dt}$$

### Example

For this example, let’s remember that _derivatives_ **can be viewed as a rate of change**. Keep in mind that the _derivative_ of a line is its slope (pending, ``gradient``), and the slope determines how much the line increases. Therefore, it makes sense that it is its rate of change.

![[derivate_chain_rule1.png]]

We can see that there are two graphs related to height. If we use age (x) to find height (y), and we can use height to find weight (z), then we can find the rate of change of weight with respect to age directly using the function composition $z(y(x))$ and the _chain rule_:

![[derivate_chain_rule2.png]]

---

## Claude Sessions
