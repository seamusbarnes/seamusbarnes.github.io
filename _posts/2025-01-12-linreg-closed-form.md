---
layout: post
title: "Why is the Closed-Form Solution to Linear Regression Inefficient?"
date: 2025-01-12 10:00:00 +0000
categories: example
tags: [machine learning, linear regression, meths]
---

Andrew Carr posted an interesting interview question on Twitter ([tweet](https://x.com/andrew_n_carr/status/1876855682529480844)):

<div style="text-align: center">
    <img src= "{{ site.baseurl }}/assets/posts/Andrew-Carr-linear-regression-question.png" style="width: 75%">
</div>

A closed-form solution is a mathematical expression that can be explicitely evaluated using a finite number of standard operations (e.g. addition, multiplication, exponentials, logarithms etc.) without requireing _iterative computation_. A common example is using the quadratic formula (a closed-form solution) to explicitly evaluate a quadratic equation *a*x<sup>2</sup> + *b*x + _c_ without iterating over the variables for _a_, _b_ and _c_. The closed-form the a linear regression problem is **B** = (**X**<sup>T</sup>**X**)<sup>-1</sup>**X**<sup>T</sup>**y**.

Linear regression on the other hand is an iterative solution that iteratively updates the **w** weights vector to minimise the following cost function J(**w**):

$$
J(w) = \frac{1}{2n} \|Xw - y\|^2
$$

where:

**X** = feature matrix

**w** = coefficients/weights matrix

**y** = target vector

**n** = number of samples

It seems naively like if we have an explicit solution that can just be calculated, why bother with this iterative solution? Using the closed-form solution causes two main problems:

1. The time-complexity for computing **X**<sup>T</sup>**X** is _O_(_mn_<sup>2</sup>, where _m_ is the number of samples and _n_ is the number of features, and tiem time complexity for computing (**X**<sup>T</sup>**X**)<sup>-1</sup> is _n_<sup>3</sup>. If your training dataset has many samples or many features, one of these time complexity's will kill you.
2. If **X** has colinear or nearly colinear features, it is referred to as ill-conditioned, this leads to zero or near-zero eigenvalues which in turn leads to numerical instability when calculating the inversion (**X**<sup>T</sup>**X**)<sup>-1</sup> because of the amplification of small floating-point rounding errors, loss of precision, and increased sensitivity to perturbation (small changes in the **X** dataset, for example with a slightly differently distributed validation set).

For these two reasons, the iterative gradient descent method is preferred for almost all use cases. Gradient descent also always reaches the optimal solution (the local minimum is also the global minimum) if it is properly formulated as it uses a quadratic cost function (mean squared error, MSE) which is a convex function.

To minimise the cost function the algorithm must calculate the gradient of the cost function with respect to each parameter, and then update that parameter based on the learning rate:

$$
J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} \left( h_\theta(x^{(i)}) - y^{(i)} \right)^2
$$

$$
\frac{\partial J(\theta)}{\partial \theta_j} = \frac{1}{m} \sum_{i=1}^m \left( h_\theta(x^{(i)}) - y^{(i)} \right) x_j^{(i)}
$$

$$
\theta_j^{(t+1)} = \theta_j^{(t)} - \alpha \frac{\partial J(\theta)}{\partial \theta_j}
$$

where:

- _J_(_θ_): The cost function, which measures the MSE between predictions and actual values.
- _m_: The number of training examples.
- _h_<sub>θ</sub>(x<sup>(i)</sup>): The predicted value for the i-th example, computed using the hypothesis function, _h_<sub>θ</sub>(_x_) = θ<sub>θ</sub> + θ<sub>1</sub>_x_<sub>1</sub> + ... + θ<sub>_n_</sub>_x_<sub>_n_</sub>.
- _y_<sup>(i)</sup>: The true value for the i-th training example.
- ∂J(θ) / ∂θ<sub>j</sub>: Partial derivative of the cost function J(θ) with respect to the parameter θ<sub>j</sub>. This represents how sensitive the cost function is to changes in θ<sub>j</sub>.
- x<sub>j</sub><sup>(i)</sup>: The value of the j-th feature for the i-th training example.
- θ<sub>j</sub><sup>(t)</sup>: The value of the parameter θ<sub>j</sub> at iteration t.
- θ<sub>j</sub><sup>(t+1)</sup>: The updated value of the parameter θ<sub>j</sub> after the (t+1)-th iteration.
- α: The learning rate, which controls the step size for updating θ<sub>j</sub>. A small α leads to slow convergence, while a large α might cause divergence.
- ∂J(θ) / ∂θ<sub>j</sub>: The gradient of the cost function with respect to θ<sub>j</sub>, which determines the direction and magnitude of the update.

<!-- ![Equation](<https://latex.codecogs.com/png.latex?J(w)%20=%20\frac{1}{2n}%20|Xw%20-%20y|^2>) -->
