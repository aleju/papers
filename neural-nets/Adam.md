# Paper

* **Title**: Adam: A Method for Stochastic Optimization
* **Authors**: Diederik Kingma, Jimmy Ba
* **Link**: https://arxiv.org/abs/1412.6980
* **Tags**: Neural Network, optimizer
* **Year**: 2015

# Summary

* What
  * They suggest a new stochastic optimization method, similar to the existing SGD, Adagrad or RMSProp.
    * Stochastic optimization methods have to find parameters that minimize/maximize a stochastic function.
    * A function is stochastic (non-deterministic), if the same set of parameters can generate different results. E.g. the loss of different mini-batches can differ, even when the parameters remain unchanged. Even for the same mini-batch the results can change due to e.g. dropout.
  * Their method tends to converge faster to optimal parameters than the existing competitors.
  * Their method can deal with non-stationary distributions (similar to e.g. SGD, Adadelta, RMSProp).
  * Their method can deal with very sparse or noisy gradients (similar to e.g. Adagrad).

* How
  * Basic principle
    * Standard SGD just updates the parameters based on `parameters = parameters - learningRate * gradient`.
    * Adam operates similar to that, but adds more "cleverness" to the rule.
    * It assumes that the gradient values have means and variances and tries to estimate these values.
      * Recall here that the function to optimize is stochastic, so there is some randomness in the gradients.
      * The mean is also called "the first moment".
      * The variance is also called "the second (raw) moment".
    * Then an update rule very similar to SGD would be `parameters = parameters - learningRate * means`.
    * They instead use the update rule `parameters = parameters - learningRate * means/sqrt(variances)`.
      * They call `means/sqrt(variances)` a 'Signal to Noise Ratio'.
      * Basically, if the variance of a specific parameter's gradient is high, it is pretty unclear how it should be changend. So we choose a small step size in the update rule via `learningRate * mean/sqrt(highValue)`.
      * If the variance is low, it is easier to predict how far to "move", so we choose a larger step size via `learningRate * mean/sqrt(lowValue)`.
  * Exponential moving averages
    * In order to approximate the mean and variance values you could simply save the last `T` gradients and then average the values.
    * That however is a pretty bad idea, because it can lead to high memory demands (e.g. for millions of parameters in CNNs).
    * A simple average also has the disadvantage, that it would completely ignore all gradients before `T` and weight all of the last `T` gradients identically. In reality, you might want to give more weight to the last couple of gradients.
    * Instead, they use an exponential moving average, which fixes both problems and simply updates the average at every timestep via the formula `avg = alpha * avg + (1 - alpha) * avg`.
    * Let the gradient at timestep (batch) `t` be `g`, then we can approximate the mean and variance values using:
      * `mean = beta1 * mean + (1 - beta1) * g`
      * `variance = beta2 * variance + (1 - beta2) * g^2`.
      * `beta1` and `beta2` are hyperparameters of the algorithm. Good values for them seem to be `beta1=0.9` and `beta2=0.999`.
      * At the start of the algorithm, `mean` and `variance` are initialized to zero-vectors.
  * Bias correction
    * Initializing the `mean` and `variance` vectors to zero is an easy and logical step, but has the disadvantage that bias is introduced.
    * E.g. at the first timestep, the mean of the gradient would be `mean = beta1 * 0 + (1 - beta1) * g`, with `beta1=0.9` then: `mean = 0.9 * g`. So `0.9g`, not `g`. Both the mean and the variance are biased (towards 0).
    * This seems pretty harmless, but it can be shown that it lowers the convergence speed of the algorithm by quite a bit.
    * So to fix this pretty they perform bias-corrections of the mean and the variance:
      * `correctedMean = mean / (1-beta1^t)` (where `t` is the timestep).
      * `correctedVariance = variance / (1-beta2^t)`.
      * Both formulas are applied at every timestep after the exponential moving averages (they do not influence the next timestep).

![Algorithm](images/Adam__algorithm.png?raw=true "Algorithm")
