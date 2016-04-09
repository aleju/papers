# Paper

* **Title**: Noisy Activation Functions
* **Authors**: Caglar Gulcehre, Marcin Moczulski, Misha Denil, Yoshua Bengio
* **Link**: http://arxiv.org/abs/1603.00391v3
* **Tags**: Neural Network
* **Year**: 2016

# Summary

![Architecture](images/DenseCap__architecture.png?raw=true "Architecture.")



--------------------

# Rough chapter-wise notes

* (1) Introduction
  * ReLU and Maxout activation functions have improved the capabilities of training deep networks.
  * Previously, tanh and sigmoid were used, which were only suited for shallow networks, because they saturate, which kills the gradient.
  * They suggest a different avenue: Use saturating nonlinearities, but inject noise when they start to saturate (and let the network learn how much noise is "good").
  * The noise allows to train deep networks with saturating activation functions.
  * Many current architectures (LSTMs, GRUs, Neural Turing Machines, ...) require "hard" decisions (yes/no). But they use "soft" activation functions to implement those, because hard functions lack gradient.
  * The soft activation functions can still saturate (no more gradient) and don't match the nature of the binary decision problem. So it would be good to replace them with something better.
  * They instead use hard activation functions and compensate for the lack of gradient by using noise (during training).
  * Networks with hard activation functions outperform those with soft ones.

* (2) Saturating Activation Functions
  * Activation Function = A function that maps a real value to a new real value and is differentiable almost everywhere.
  * Right saturation = The gradient of an activation function becomes 0 if the input value goes towards infinity.
  * Left saturation = The gradient of an activation function becomes 0 if the input value goes towards -infinity.
  * Saturation = A activation function saturates if it right-saturates and left-saturates.
  * Hard saturation = If there is a constant c for which for which the gradient becomes 0.
  * Soft saturation = If there is no constant, i.e. the input value must become +/- infinity.
  * Soft saturating activation functions can be converted to hard saturating ones by using a first-order Taylor expansion and then clipping the values to the required range (e.g. 0 to 1).
  * A hard activating tanh is just `f(x) = x`. With clipping to [-1, 1]: `max(min(f(x), 1), -1)`.
  * The gradient for hard activation functions is 0 above/below certain constants, which will make training significantly more challenging.
  * hard-sigmoid, sigmoid and tanh are contractive mappings, hard-tanh for some reason only when it's greater than the threshold.
  * The fixed-point for tanh is 0, for the others !=0. That can have influences on the training performance.

* (3) Annealing with Noisy Activation Functions
  * Suppose that there is an activation function like hard-sigmoid or hard-tanh with additional noise (iid, mean=0, variance=std^2).
  * If the noise's `std` is 0 then the activation function is the original, deterministic one.
  * If the noise's `std` is very high then the derivatives and gradient become high too. The noise then "drowns" signal and the optimizer just moves randomly through the parameter space.
  * Let the signal to noise ratio be `SNR = std_signal / std_noise`. So if SNR is low then noise drowns the signal and exploration is random.
  * By letting SNR grow (i.e. decreaseing `std_noise`) we switch the model to fine tuning mode (less coarse exploration).
  * That is similar to simulated annealing, where noise is also gradually decreased to focus on better and better regions of the parameter space.

* (4) Adding Noise when the Unit Saturate
  * This approach does not always add the same noise. Instead, noise is added proportinally to the saturation magnitude. More saturation, more noise.
  * That results in a clean signal in "good" regimes (non-saturation, strong gradients) and a noisy signal in "bad" regimes (saturation).
  * Basic activation function with noise: `phi(x, z) = h(x) + (mu + std(x)*z)`, where `h(x)` is the saturating activation function, `mu` is the mean of the noise, `std` is the standard deviation of the noise and `z` is a random variable.
  * Ideally the noise is unbiased so that the expectation values of `phi(x,z)` and `h(x)` are the same.
  * `std(x)` should take higher values as h(x) enters the saturating regime.
  * To calculate how "saturating" a activation function is, one can `v(x) = h(x) - u(x)`, where `u(x)` is the first-order Taylor expansion of `h(x)`.
  * Empirically they found that a good choice is `std(x) = c(sigmoid(p*v(x)) - 0.5)^2` where `c` is a hyperparameter and `p` is learned.
  * (4.1) Derivatives in the Saturated Regime
    * For values below the threshold, the gradient of the noisy activation function is identical to the normal activation function.
    * For values above the threshold, the gradient of the noisy activation function is `phi'(x,z) = std'(x)*z`. (Assuming that z is unbiased so that mu=0.)
  * (4.2) Pushing Activations towards the Linear Regime
    * 

