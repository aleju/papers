# Overview

Title: Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift
Authors: Sergey Ioffe, Christian Szegedy
Link: http://arxiv.org/abs/1502.03167
Tags: Neural Network, Performance, Covariate Shift, Regularization
Year: 2015

# Summary

* Batch Normalization (*BN*) is a special normalization method for neural networks.
* In neural networks, the inputs to each layer depend on the outputs of all previous layers.
* The distributions of these outputs can change during the training. Such a change is called a *covariate shift*.
* If the distributions stayed the same, it would simplify the training. Then only the parameters would have to be readjusted continuously (e.g. mean and variance for normal distributions).
* If using sigmoid activations, it can happen that one unit saturates (very high/low values). That is undesired as it leads to vanishing gradients for all units below in the network.

* BN fixes the means and variances of layer inputs to specific values (zero mean, unit variance).
* That accomplishes:
  * No more covariate shift.
  * Fixes problems with vanishing gradients due to saturation.
* Effects:
  * Networks learn faster. (As they don't have to adjust to covariate shift any more.)
  * Optimizes gradient flow in the network. (As the gradient becomes less dependent on the scale of the parameters and their initial values.)
  * Higher learning rates are possible. (Optimized gradient flow reduces risk of divergence.)
  * Saturating nonlinearities can be safely used. (Optimized gradient flow prevents the network from getting stuck in saturated modes.)
  * BN reduces the need for dropout. (As it has a regularizing effect.)

* How BN works:
  * BN normalizes layer inputs to zero mean and unit variance. That is called *whitening*.
  * Naive method: Train on a batch. Update model parameters. Then normalize. Doesn't work: Leads to exploding biases while distribution parameters (mean, variance) don't change.
  * A proper method has to include the current example *and* all previous examples in the normalization step.
  * This leads to calculating in covariance matrix and its inverse square root. That's expensive. The authors found a faster way.

* (3) Normalization via Mini-Batch Statistics
  * Each feature (component) is normalized individually. (Due to cost, differentiability.)
  * Normalization according to: `componentNormalizedValue = (componentOldValue - E[component]) / sqrt(Var(component))`
  * Normalizing each component can reduce the expressitivity of nonlinearities. Hence the formula is changed so that it can also learn the identiy function.
  * Full formula: `newValue = gamma * componentNormalizedValue + beta` (gamma and beta learned per component)
  * E and Var are estimated for each mini batch.
  * BN is fully differentiable. Formulas for gradients/backpropagation are at the end of chapter 3 (page 4, left).
