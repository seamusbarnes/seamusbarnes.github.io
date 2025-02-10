---
layout: post
title: "What I Learned From a Postdoc Interview Rejection"
date: 2025-01-09 10:00:00 +0000
categories: til
tags: [personal, career, coding interviews, postdoc, python, linear regression]
excerpt:
  "Doing bits of coding here or there is not enough, you need to lock in and
  have the fundamentals at your fingertips."
---

## Context

I start working on my career change almost a year ago when I quit my perfectly fine but not very inspiring engineering job. Before that point I had spent about seven years studying and then working as a photonics researcher, but decided to throw away that highly specialised skillset to pursue a career in AI<sup>TM</sup>, because why not?

It has been a humbling experience teaching myself coding, maths, machine learning, etc. from scratch, and then competing in the job market against 21-year-old whippersnappers fresh out of university with their CS degrees and half a decade of locked in code grinding under their belts. I started applying for positions in the final quarter of 2024, maybe I have submitted close to 100 real applications, and I got one interview. I had the interview in December and it went OK (3.6 roentgen, not great, not terrible) and just received a rejection email (oh woe is me). This TIL is about what I learned from that interview.

The interview was a simple format and fell mainly into two sections: 1) _behavioral questions_ (tell me about a time when blah blah blah), 2) _coding questions_. Despite the fact that I had read up quite a bit on the topic (AI for quantum science and computing), I had failed to seriously prepare appropriately for either section.

## 1. The Behavioral Questions

For the behavioral questions my lack of preparedness translated to a default strategy of basically winging it. This worked in the past for previous "application rounds" because I had explicitly prepared answers/stories and had many more interviews during which I got better and more confident with answers. This time I was so out of practice winging it seriously didn't work. The second problem was with the coding question

## 2. The Coding Questions

The coding questions went equally poorly. I had done lots of coding question exercises (bigup my man Neetcode), and studied ML basics, but when I was asked to implement a simple linear regression model on randomly generated data (from something like y = *a*x<sub>2</sub> + *b*x<sub>1</sub> + *c*x<sub>0</sub> + _ε_, where _a_ and _b_ were variables and _ε_ was a noise term), I found myself completely floundering. I had to understand the question first, which was phrased in a deliberately ambiguous way, and then use `numpy`, `scikit-learn` and `matplotlib` to generate the data, train the model and then do some visualisations. This should have been basic stuff, I had done all of this in the past, but it wasn't at my fingertips.

I also failed afterwards to properly explain exactly how linear regression worked, and even resorted to using chatgpt in a very naive way for the coding section (basically spamming different code into ChatGPT and trying different things mindlessly, the interviewer was aware of this and encouraged me to use any tools available including llms, but I did this thoughtlessly and chaotically which must have looked bad). I mixed up the different ways linear parameters can be calculated, on the one hand solving the closed-form formula, and on the other hand using gradient descent to minimise the MSE.

The main takeaway from the coding section has to be that I haven't really internalised lots of the knowledge I thought I was learning. I've watched a lot of videos and done a bit of coding projects here and there, but it's still nowhere near internalised. I can do alright on an "open book" test, but to be serious about this stuff I need to be able to do it "closed book".

## The Code I Should Have Written

The following should be taken with a healthy dose of salt. It is merely an example of one concise way to demonstrate linear regression on a polynomially characterised dataset. This code doesn't split the dataset (_x_ and _y_noisy_) into training and validation datasets, which is normal practice when training and evaluating machine learning models. I ran this in an `Jupyter` notebook.

_Concise version_

```python
# imports
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

# prepare data and plot true polynomial and noisy generated data together
min_x = -5
max_x = 5
num_samples = 100
a = 2
b = 3
c = 0

x = np.linspace(min_x, max_x, num_samples)
y = a * x**2 + b * x + c
y_noisy = y + np.random.normal(loc=0, scale=5.0, size=num_samples)

plt.figure(figsize=(9, 3))
plt.plot(x, y, color='red', label='True Function')
plt.scatter(x, y_noisy, label='Noisy Data')
plt.xlim((min_x, max_x))
plt.legend()
plt.title("True polynomial data")
plt.show()

# pre-process features (1D x data) so it is compatible with the PolynomialFeatures and LinearRegression classes that expect input data in a 2D format, where each row represents a sample and each column represents a feature or its transformation
x_reshaped = x.reshape(-1, 1)
poly = PolynomialFeatures(degree=2, include_bias=False)
x_poly = poly.fit_transform(x_reshaped)

# train the LinearRegression model, make predictions to generate y_pred, and print the mse, learned polynomial coefficients and learned intercept
model = LinearRegression()
model.fit(x_poly, y_noisy)

y_pred = model.predict(x_poly)

mse = mean_squared_error(y, y_pred)
print(f"Mean squared error: {mse:.4f}")
print(f"Learned coefficients: {model.coef_[0]:.4f}, {model.coef_[1]:.4f}")
print(f"Learned intercept: {model.intercept_:.4f}")

# plot the true polynomial and the polynomial with the learned coefficients and intercept on the same graph to visually compare the quality of the model
plt.figure(figsize=(9, 3))
plt.plot(x, y, linestyle='-', color='red', label='True polynomial without noise')
plt.plot(x, y_pred, linestyle='--', color='blue', label='Predicted polynomial')
plt.legend()
plt.title('True polynomial and predicted polynomial')
plt.show()
```
