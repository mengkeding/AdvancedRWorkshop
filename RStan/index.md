---
title       : RStan
subtitle    : UCI Data Science Initiative
author      : Sepehr Akhavan
job         : Dept. of Statistics
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : mathjax            # {mathjax, quiz, bootstrap}
logo     : logo.png
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
github:
  user: UCIDataScienceInitiative
  repo: AdvancedRWorkshop
---

## Agenda


+ Brief introduction to Bayesian Analysis
+ Available software for Bayesian Analysis
+ Bayesian Analysis using RStan
+ Examples

---

## Bayesian Analysis

Baye's Theorem:

$$ P(\theta|Y) = \frac{P(Y | \theta) P(\theta)}{P(Y)}$$

Where:
+ $\theta$ : unknown parameter(s)
+ $Y$ : observed data

We can write the formula above as:
$$ P(\theta|Y) \propto P(Y | \theta) P(\theta) = P(\theta, Y)$$

__Goal__ : To characterize the posterior distribution $P(\theta|Y)$

---

## Bayesian Analysis

Posterior distribution $P(\theta|Y)$ :
  + 1) may have a closed form (ex. conjugate prior)
  + 2) USUALLY doesn't have a closed form and is not analytically tractable! 
  
if (2), How to generate samples from $P(\theta|Y)$ ?

  + solution 1) MCMC: a class of algorithms for sampling from a probability distribution based on constructing a Markov chain that has the desired distribution as its equilibrium distribution. 
    + ref:[An Introduction to MCMC for Machine Learning](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.13.7133&rep=rep1&type=pdf)
  
  + solution 2) Variational Bayes: Approximate the posterior with a simpler distribution.

---

## Bayesian Analysis

Some examples of MCMC methods:

  + Metropolis–Hastings algorithm
  + Gibbs sampling
  + Slice sampling
  + Reversible-jump
  + Hamiltonian Monte Carlo
    + [MCMC Using Hamiltonian Dynamics] (http://www.mcmchandbook.net/HandbookChapter5.pdf)

---

## Some Bayesian Analysis Packages:

1) WinBUGS/OpenBUGS:
  + http://www.mrc-bsu.cam.ac.uk/software/bugs/


2) JAGS:
  + http://mcmc-jags.sourceforge.net


3) Stan:
  + http://mc-stan.org

---

## What is Stan?

+ "a modeling language in which statisticians could write their models in familiar notation that could be transformed to efficient C++ code and then compiled into an efficient executable program."

+ Stan uses HMC method for sampling as opposed to Gibbs sampling.

+ Stan automatically adapt the number of steps during HMC sampling using No-U-Turn (NUTS) sampler:
  + ref: Hoffman and Gelman, 2011, 2014

+ Stan's interfaces include: RStan, PyStan, CmdStan, ...

+ to install stan, please check:
  + https://github.com/stan-dev/rstan/wiki/RStan-Getting-Started

---

## Where to get help?

1) Stan User's Guide & Reference Manual:
  + Current version: 2.8.0
  + You can access it [here] (https://github.com/stan-dev/stan/releases/download/v2.8.0/stan-reference-2.8.0.pdf)

2)  Stan User's group:
  + https://groups.google.com/forum/#!forum/stan-users

---

## Simple Posterior Example

+ Consider an example of flipping a coin N times where $P(head) = \theta$:
$$P(Y | \theta) \propto \theta^Y (1 - \theta)^{(N - Y)}$$

+ Now, suppose $\theta$ (probability of head) is unknown. We can assume a prior of the form:
$$\theta \sim Beta(\alpha, \beta)$$

+ Under this setting, posterior of $\theta$ would be of the form:
$$P(\theta|Y) \propto P(Y | \theta) P(\theta)$$

+ It's then easy to show:
$$P(\theta|Y) \propto \theta^{(\alpha + Y - 1)} (1 - \theta)^{(\beta + (N - Y) - 1)}$$
$$P(\theta|Y) \propto Beta\big(\alpha + Y, \beta + (N - Y) \big)$$

---

## Simple Posterior Example

+ Life is easy when posterior distribution is tractable :)


```r
N <- 200
theta.true <- 0.4
Y <- rbinom(1, size=N, prob = 0.4)
alpha <- 2
beta <- 2

PostSample <- rbeta(10000, shape1 = alpha + Y,  shape2 = beta + (N - Y))

# Posterior Mean:
mean(PostSample)
```

```
## [1] 0.3778661
```

```r
# 95% CR:
quantile(PostSample, probs = c(0.025, 0.975))
```

```
##      2.5%     97.5% 
## 0.3131006 0.4442364
```

---

## Simple Posterior Example in Stan

There are 3 steps needed to implement this model in Stan:

1) Writing the model in Stan

2) Initializing the data for the model

3) Running the model

---

## Simple Posterior Example in Stan (Stan Model)


```r
data {
  int<lower = 1> N;
  int<lower = 0, upper = N> Y;
  real<lower = 0> alpha;
  real<lower = 0> beta;
}
parameters {
  real<lower = 0, upper = 1> theta;
}
model {
  theta ~ beta(alpha, beta);
  Y ~ binomial(N, theta);
}
```

+ Good practice to save your stan model as a .txt or .stan into a file.

---

## Simple Posterior Example in Stan


```r
# We already have N, Y,alpha, and beta
data = list(N = N, Y = Y, alpha = alpha, beta = beta)
fit <- stan("SimpleCoinEx.stan", data = data, chains = 1, iter = 1000, warmup = 200)
summary(fit)
```


---

## Bayesian Modeling with Stan (R section)

+ stan(): does all of the work of fitting a Stan model and returning the results. It includes:
  + 1) translating the Stan model to C++ code
  + 2) Then, the C++ code is compiled into a binary shared object, which is loaded into the current R session 
  + 3) Finally, samples are drawn and wrapped in an object
  
  
+ stan(): can also be used to sample again from a fitted model under different settings (e.g., different iter).
  + To do so, fit argument should be provided


---

## Bayesian Modeling with Stan (R section)

Useful functions to try on an stan fit:

  + summary()
  
  + print()
  
  + traceplot()
  
  + extract()
  
  + launch_shinystan{shinystan}


---

## Bayesian Modeling with Stan (Stan Model)

The rest of the workshop will be on details of writing stan models. In particular, we will learn more on:

+ Stan's Program Blocks

+ Data Types, Variable Declaration, and Statements

+ Log Likelihood Specification

+ User-Defined Functions

+ Examples 

---

## Stan's Program Blocks 


```r
functions{
}
data {
}
transformed data {
}
parameters {
}
transformed parameters {
}
model {
}
generated quantities{ 
}
```

---

### function block:

+ This optional block contains user-defined functions (if there is any)

+ more on this later

---

### data Block

+ All variables that are read in as data are declared here

+ Data are read only once!

+ The data block does not allow statements

+ Input data should jibe with the declared data in this block:
  + within the same range
  + having the same size


---

### transformed data Block

+ Defining variables that are fixed when running the program

+ No reading from external sources

+ Variables defined earlier in the data block may be used here


---

### parameters Block

+ Variables defined in this block are sampled

+ Variables defined as parameters cannot be directly assigned


---

### transformed parameters Block

+ There is no need to perform a direct sampling for variables declared under this block

+ Variables that are defined in terms of data or transformed data only (fixed) should be defined in the transformed data block. Those variables are illegal under this block

+ Variables defined here will be written as part of the output

---

### model Block

+ likelihood (the model/distribution the data come from) is defined under this block

+ priors on the parameters are also defined in this block

---

### generated quantities Block

+ This block is executed only after a sample has been generated

+ Therefore, nothing in the generated quantities block affects the sampled parameter values

+ Some applications of this block are:
  
  + to calculate posterior expectations
  
  + transforming parameters for reporting
  
  + generating predictions for new data

---

## Data Types and Variable Declarations:

+ Variables should have a declared data type. Stan's data types include:

+ 1) Primitive Data Types:
  + 1.1) real
  + 1.2) int
  + 1.3) constrained by using lower or upper
  
+ 2) Compound Data Types:
  + 2.1) vector (of real values)
  + 2.2) row_vector (of real values)
  + 2.3) matrix (of real values)
  + 2.4) constrained include: simplex, unit_vector, ordered, positive_ordered, corr_matrix, cov_matrix, cholesky_factor_cov, cholesky_factor_cor
  
+ 3) Arrays:
  + 3.1) three-dimensional arrays of integers
  + 3.2) four-dimensional arrays of row_vectors


---

## Data Types and Variable Declarations:


```r
# primitive:
int<lower = 1> N;
real<lower = 0, upper = 1> theta;

# vector:
vector[3] myVec;
vector<lower = 0>[3] myConsVec;
simplex[5] theta;

# matrix:
matrix[3,3] A;
corr_matrix[3] Sigma;
cholesky_factor_corr[5] L;

# array:
int n[5]; // n is an array of 5 integers
real a[3,4];
vector[7] mu[3]; // 3-dimensional array of vectors of length 7
```


---

## Statements:

