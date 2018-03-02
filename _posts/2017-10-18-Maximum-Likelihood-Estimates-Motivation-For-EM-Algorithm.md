---
layout: post
type: blog
title: Maximum Likelihood Estimates - Motivation for EM algorithm
comments: true
mathjax: true
---

## Introduction
To solve any data science problem, first we obtain a dataset, do exploration on it and then, guided by the findings, we try to come up with a model to tackle the problem. Once all of that is done, our next task is to find a way to estimate the parameters of the model based on the dataset we have, so that we can make predictions on unseen data. In this post, we will learn about how we can learn the parameters of the model using Maximum Likelihood approach which has a very simple premise: find parameters that maximize the likelihood of the observed data. Through that, I would motivate the Expectation-Maximization (EM) algorithm which is considered to be an important tool in statistical analysis.

## Maximum Likelihood approach in Logistic Regression
Now, let's understand Maximum Likelihood approach in the context of Logistic Regression. Suppose, we have a dataset where we denote predictor variables with $X \in R^d$ and the target variable with $Y \in \\{0,1\\}$. The model could be depicted graphically as following:

<center>
<img src="/images/mle/logistic_model.JPG" width="600" height ="300"/>
</center>

As we know, prediction probability of target variable in logistic regression is given by a sigmoid function like following:
<center>
$P(Y = 1 | X = x) = \sigma(w.x) = \frac{1}{1 + exp(-w.x)}$
</center>
Based on whether 1 or 0 has more probability of occuring based on the predictor variables, we output our prediction. But, how do we appropriate $w$ such that the error is minimized?

To that end, we will use the Maximum Likelihood approach where we'll try to find the $w$ which maximizes the likelihood of the observed data. For mathematical convenience, we'll consider try to maximize the log of the likelihood. Likelihood of data, parameterized by $z$, can be written as: 

<center>
$L(w) = log P(\text{data}) = log \Pi_{i=1}^N P(Y = y_i | X = x_i)$
</center>
Here the data, $\\{x_i,y_i\\}^i$, is represented in terms of multiplication of conditional probabilities $P(Y = y_i | X = x_i)$ assuming data samples are independently and identically distributed (so called the [i.i.d assumption](https://en.wikipedia.org/wiki/Independent_and_identically_distributed_random_variables)). 
<center>
$ \implies L(w) = \sum_{i=1}^N log  P(Y = y_i | X = x_i) = \sum_{i=1}^N log\left[\sigma(w.x_i)^{y_i} . \sigma(-w.x_i)^{1 - y_i} \right]$
</center>
where we've written $P(Y = y_i | X = x_i)$ in a general form.
<br/>
<center>
$L(w) = \sum_{i=1}^N \left[ y_i. log\sigma(w.x_i) + (1 - y_i). log\sigma(-w.x_i) \right]$ --- (A)
</center>

At this point, we should note that log likelihood $L(w)$ breaks down conveniently into per-instance form. Since, there's no coupling between the parameters, optimization can be done easily and we'll see later why this is a good thing. Since $L(w)$ is a function of $w$, we don't have any closed form solution to equation (A). Thus, we would have to use iterative optimization methods like gradient ascent or newton's method to find $w$. An update for gradient ascent method would look like:
<center>
$w = w + \eta.\left( \sum_{i=1}^N [y_i - \sigma(w.x_i)].x_i \right)$ --- (B)
</center>
where $\eta$ is an appropriate learning rate. We repeat (B) until convergence. The final value of $w$ we opt for is called maximum likelihood estimate.

### Case of latent variables in model 
One detail I didn't point out about the Logistic Regression model was that all the predictor variables of the model are observed. However, there can be problem which require to have latent (unobserved) predictor variables in the model. One great example for this kind of model is the [Hidden Markov model](https://en.wikipedia.org/wiki/Hidden_Markov_model) where the actual state is not directly visible, but the output, dependent on the state, is visible. This model has applications in speech, handwriting, gesture recognition, part-of-speech tagging, musical score following and other areas. But, does it make any difference in estimation of parameters if we have hidden variables in our model? Let's see.

## EM algorithm to the Rescue
It turns out, estimating model parameters does gets a little tricky if latent variables are involved. Let's see why. Let $V$ be the observed variables (this includes the target variable) in the model and $Z$ be the latent variables in the model. As per the maximum likelihood approach, our objective to maximize would be:
<center>
$L(w) = log P(\text{data}) = log \Pi_{i=1}^N P(V_i) = \sum_{i=1}^N log P(V_i) = \sum_{i=1}^N log \sum_{h \in Z_i} P(V_i, h)$
</center>
This can be written in form of conditional probabilities as following:
<center>
$L(w) = \sum_{i=1}^N log \sum_{h \in Z_i} \Pi_{j} P(X_j | \text{Pa}(X_j))$
</center>
where $X_j \in \\{V_i, Z_i\\}$ and $\text{Pa}(X_j)$ represents parent of $X_j$ in [belief network](http://artint.info/html/ArtInt_148.html) of the model.