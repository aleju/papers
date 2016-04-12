# Paper

* **Title**: Auto-Encoding Variational Bayes
* **Authors**: Diederik P Kingma, Max Welling
* **Link**: http://arxiv.org/abs/1312.6114
* **Tags**: Neural Network, Autoencoder, VAE
* **Year**: 2014

# Summary

![Architecture](images/DenseCap__architecture.png?raw=true "Architecture.")



--------------------

# Rough chapter-wise notes

* (1) Introduction
  * In a directed probabilistic model we latent/hidden variables with intractable posterior distributions.
  * Variational Bayesian Approach (VB): Optimize an approximation to these posteriors.
  * Common VB method: mean-field, which (in the general case) also has intractable posteriors.
  * Their SGVB estimator (Stochastic Gradient Variational Bayes):
    * Is an unbiased estimator of the variational lower bound.
    * Is based on a reparameterization of the variational lower bound.
    * Is differentiable. Can be optimized with SGD.
    * Can be used for approximate posterior inference (in almost any model, must(?) have continuous latent variables/parameters).
  * AEVB algorithm (Auto-Encoding Variational Bayes):
    * Works with iid datasets that have continuous latent variables.
    * Optimizes a recognition model (the approximation to the posterior?).
    * That optimization is done using the SGVB estimator.
    * The recognition model can be used for efficient learning and approximate inference using ancestral sampling (apparently another name for forward sampling?).
    * If the recognition model is a neural net, it is called "variational auto-encoder".

* (2) Method
  * Assumptions:
    * iid dataset.
    * Continuous latent variables per datapoint.
    * Inference via MLE or MAP on the parameters.
    * Variational inference on the latent variables.
  * Strategy: Derive lower bound estimator (for a variety of directed graphical models with continuous latent variables).
  * (2.1) Problem scenario
    * Assume we have a dataset.
    * Assume that each entry in that dataset has one observed variable `x` and one unobserved variable `z`.
    * Assume that `z` has no parents and just follows the prior `p(z)`.
    * Assume that `x` has `z` as its parent and follows the likelihood `p(x|z)`.
    * Assume that `p(z)` and `p(x|z)` come from parametric families of distributions (e.g. normal distribution), which have the parameters `theta`.
    * Assume that the probability density functions (PDFs) of `p(z)` and `p(x|z)` are both differentiable (almost) everywhere (with respect to `theta` and `z`).
    * No simpifying assumption for the marginal probabilities or conditional probabilities.
    * The algorithm is supposed to work even when (1) p(x) and/or p(z|x) are intractable with standard VB methods and (2) when we have huge datasets that require processing in minibatches.
    * Interesting tasks to solve:
      * (Efficient) MLE and MAP estimation of parameters `theta`.
      * Inference of p(z|x). Examples: Dimensionality reduction.
      * Inference of p(x). Examples: Denoising, inpainting, super-resolution.
    * They introduce a recognition model q(z|x) that approximates p(z|x) (e.g. from an image to latent variables like rotation, scaling, ...).
    * q(z|x) can be viewed as an encoder that takes an x and produces a distribution (e.g. Gaussian) over z.
    * p(x|z) can be viewed as a decoder that takes a z and produces a distribution (e.g. Gaussian) over x.
    * z can be viewed as the code layer and any configuration of z as a code.
  * (2.2) The variational bound
    * The probability of the whole dataset is `p(X) = p(x1)*p(x2)*...`.
    * Or with log: `log p(x) = log p(x1) + log p(x2) + ...`.
    * The probability of a single datapoint is `log p(xi) = KL(q(z|xi) || p(z||xi)) + L(theta, phi; xi)` (no real explanation where this suddenly comes from).
    * KL(a||b) is the KL-divergence between p(z|x) and p(z|x)'s approximation, i.e. q(z|x). The KL-Divergence measures the difference between the two probability distributions. It is always `>=0`. It is close to 0 if the approximation is very close to the real distribution.
    * `L(theta, phi; xi)` is the variational lower bound. It is defined as `-KL(q(z|xi) || p(z)) + E[log p(xi|z)]`. `theta` are the parameters of p(x|z) and p(z). `phi` are the paramters of p(z|x).
    * We want to find optimal parameters for `theta` and `phi`. Usually one would use Monte Carlo methods for that, but they lead to too much variance.
  * (2.3) The SGVB estimator and AEVB algorithm
    * This section discusses a method to estimate the lower bound and its derivatives wrt to the parameters.
    * Instead of using the probability distribution q(z|x) (e.g. from an image to the latent variables) we use the estimator z=g(epsilon, x) where epsilon is a noise variable (e.g. from an image and some noise to the latent variables).
    * Using Monte Carlo Methods, one can now approximate the variational lower bound: `La(theta, phi; xi) = 1/L sum log p(xi, g(epsilon, xi)) - log q(g(epsilon, xi)|xi)`
    * The given formula is for the full lower bound, i.e. for `L = -KL(q(z|xi)||p(z)) + E[log p(xi|z)] = -ComplexitPenalty + ReconstructionError`. The KL-Divergence can usually be computed analytically, so sampling it would be wasteful. That leads to the formular `Lb = -KL(q(z|xi)||p(z)) + Sample[log p(xi|zi)] = -KL(q(z|xi)||p(z)) + 1/L sum (log p(xi|zi))`.
    * For a dataset with N examples and a batch size of M, the lower bound can be approximated via `N/M sum L(theta, phi; xi)`.
    * The number of required samples L can be reduced to one if the batch size is high enough (>= 100).
    * Stochastic Gradient Descent can be used to approximate the lower bound.
    * The lower bound approximation is reminiscent of auto-encoders, which also usually have an objective function of the form `ReconstructionError - ComplexityPenalty`, i.e. here `E[log p(xi|z)] - KLDivergence`.
    * The basic principle of a VAE is: Take an input example xi, then generate some noise, then use g(xi, noise) to generate a sample zi of the latent variables, then take zi and convert it via p(xi|zi) back to the input, resulting in some reconstruction error.
  * (2.4) The reparameterization trick
    * 

