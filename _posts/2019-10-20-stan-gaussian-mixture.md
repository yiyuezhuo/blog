---
layout: post
title: Stan and gaussian mixture(WIP)
date: 2019-10-20 21:37
category: tech
---

When I written my undergraduate thesis, I had played with stan and found it very interesting, especially how it explain the sampling "statement":

> 7.4. Sampling Statements
>
> Stan supports writing probability statements also in sampling notation, such as
>
> `y ~ normal(mu,sigma);`
>
> The name “sampling statement” is meant to be suggestive, not interpreted literally.
Conceptually, the variable y, which may be an unknown parameter or known,
modeled data, is being declared to have the distribution indicated by the right-hand
side of the sampling statement.
>
> Executing such a statement does not perform any sampling. In Stan, a sampling
statement is merely a notational convenience. The above sampling statement could
be expressed as a direct increment on the total log probability as
>
> `target += normal_lpdf(y | mu, sigma);`
>
>In general, a sampling statement of the form
>
> `y ~ dist(theta1, ..., thetaN);`
>
> involving subexpressions y and theta1 through thetaN (including the case where N
is zero) will be well formed if and only if the corresponding assignment statement is
well-formed. For densities allowing real y values, the log probability density function
is used,
>
> `target += dist_lpdf(y | theta1, ..., thetaN);`
>
> For those restricted to integer y values, the log probability mass function is used,
>
> `target += dist_lpmf(y | theta1, ..., thetaN);`
>
>This will be well formed if and only if `dist_lpdf(y | theta1, ..., thetaN)` or
`dist_lpmf(y | theta1, ..., thetaN)` is a well-formed expression of type `real`

It's exactly how a bayesian network is specified, using a pretty stan grammar.

Stan have some drawbacks such as slow compiling time and a non-trivial programming language instead of a more familiar language to user such as Python(pyro,pymc3,Edward), Julia(Turing.jl) or R.

With this definition of sampling, we can express model which is unable to sample in fact. For example, Gaussian Mixture specified in [1]:

```stan
data{
    int<lower=0> N; // number of data points in dataset
    int<lower=0> K; // number of mixture components
    int<lower=0> D; // dimension
    vector[D] x[N]; // observations
}

transformed data {
    vector<lower=0>[K] alpha0_vec;
    for (k in 1:K){             // convert the scalar dirichlet prior 1/K
        alpha0_vec[K] <- 1.0/K; // to a vector
    }
}

parameters{
    simplex[K] theta; // mixing proportions
    vector[D] mu[K]; // locations of mixture components
    vector<lower=0>[D] sigma[K]; // standard deviations of mixture components
}

model{
    // prior
    theta ~ dirichlet(alpha0_vec);
    for( k in 1:K){
        mu[k] ~ normal(0.0, 1.0/sigma[k]);
        sigma[k] ~ inv_gamma(1.0, 1.0);
    }

    // likelihood
    for (n in 1:N){
        real ps[K];
        for (k in 1:K){
            ps[k] <- log(theta[k]) + normal_log(x[n], mu[k], sigma[k]);
        }
        increment_log_prob(log_sum_exp(ps))
    }
}
```

Today, I have read some papers about Bayesian statistical ( or more precious, so called deep bayes), and realized stan being a baseline tool to compare with novel models. You can find HMC and ADVI powered by stan in many tables listed by those paper.

## References

1. Blei, David M., Alp Kucukelbir, and Jon D. McAuliffe. "Variational inference: A review for statisticians." Journal of the American Statistical Association 112.518 (2017): 859-877.