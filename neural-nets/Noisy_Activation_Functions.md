# Paper

* **Title**: Noisy Activation Functions
* **Authors**: Caglar Gulcehre, Marcin Moczulski, Misha Denil, Yoshua Bengio
* **Link**: http://arxiv.org/abs/1603.00391v3
* **Tags**: Neural Network
* **Year**: 2016

# Summary

* What
  * Certain activation functions, mainly sigmoid, tanh, hard-sigmoid and hard-tanh can saturate.
  * That means that their gradient is either flat 0 after threshold values (e.g. -1 and +1) or that it approaches zero for high/low values.
  * If there's no gradient, training becomes slow or stops completely.
  * That's a problem, because sigmoid, tanh, hard-sigmoid and hard-tanh are still often used in some models, like LSTMs, GRUs or Neural Turing Machines.
  * To fix the saturation problem, they add noise to the output of the activation functions.
  * The noise increases as the unit saturates.
  * Intuitively, once the unit is saturating, it will occasionally "test" an activation in the non-saturating regime to see if that output performs better.

* How
  * The basic formula is: `phi(x,z) = alpha*h(x) + (1-alpha)u(x) + d(x)std(x)epsilon`
  * Variables in that formula:
    * Non-linear part `alpha*h(x)`:
      * `alpha`: A constant hyperparameter that determines the "direction" of the noise and the slope. Values below 1.0 let the noise point away from the unsaturated regime. Values <=1.0 let it point towards the unsaturated regime (higher alpha = stronger noise).
      * `h(x)`: The original activation function.
    * Linear part `(1-alpha)u(x)`:
      * `u(x)`: First-order Taylor expansion of h(x).
        * For sigmoid: `u(x) = 0.25x + 0.5`
        * For tanh: `u(x) = x`
        * For hard-sigmoid: `u(x) = max(min(0.25x+0.5, 1), 0)`
        * For hard-tanh: `u(x) = max(min(x, 1), -1)`
    * Noise/Stochastic part `d(x)std(x)epsilon`:
      * `d(x) = -sgn(x)sgn(1-alpha)`: Changes the "direction" of the noise.
      * `std(x) = c(sigmoid(p*v(x))-0.5)^2 = c(sigmoid(p*(h(x)-u(x)))-0.5)^2`
        * `c` is a hyperparameter that controls the scale of the standard deviation of the noise.
        * `p` controls the magnitude of the noise. Due to the `sigmoid(y)-0.5` this can influence the sign. `p` is learned.
      * `epsilon`: A noise creating random variable. Usually either a Gaussian or the positive half of a Gaussian (i.e. `z` or `|z|`).
  * The hyperparameter `c` can be initialized at a high value and then gradually decreased over time. That would be comparable to simulated annealing.
  * Noise could also be applied to the input, i.e. `h(x)` becomes `h(x + noise)`.

* Results
  * They replaced sigmoid/tanh/hard-sigmoid/hard-tanh units in various experiments (without further optimizations).
  * The experiments were:
    * Learn to execute source code (LSTM?)
    * Language model from Penntreebank (2-layer LSTM)
    * Neural Machine Translation engine trained on Europarl (LSTM?)
    * Image caption generation with soft attention trained on Flickr8k (LSTM)
    * Counting unique integers in a sequence of integers (LSTM)
    * Associative recall (Neural Turing Machine)
  * Noisy activations practically always led to a small or moderate improvement in resulting accuracy/NLL/BLEU.
  * In one experiment annealed noise significantly outperformed unannealed noise, even beating careful curriculum learning. (Somehow there are not more experiments about that.)
  * The Neural Turing Machine learned far faster with noisy activations and also converged to a much better solution. 


![Influence of alphas](images/Noisy_Activation_Functions__alphas.png?raw=true "Influence of alphas.")

*Hard-tanh with noise for various alphas. Noise increases in different ways in the saturing regimes.*


![Neural Turing Machine results](images/Noisy_Activation_Functions__ntm.png?raw=true "Neural Turing Machine results.")

*Performance during training of a Neural Turing Machine with and without noisy activation units.*

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
    * In saturated regimes, one would like to have more of the noise point towards the unsaturated regimes than away from them (i.e. let the model try often whether the unsaturated regimes might be better).
    * To achieve this they use the formula `phi(x,z) = alpha*h(x) + (1-alpha)u(x) + d(x)std(x)epsilon`
      * `alpha`: A constant hyperparameter that determines the "direction" of the noise and the slope. Values below 1.0 let the noise point away from the unsaturated regime. Values <=1.0 let it point towards the unsaturated regime (higher alpha = stronger noise).
      * `h(x)`: The original activation function.
      * `u(x)`: First-order Taylor expansion of h(x).
      * `d(x) = -sgn(x)sgn(1-alpha)`: Changes the "direction" of the noise.
      * `std(x) = c(sigmoid(p*v(x))-0.5)^2 = c(sigmoid(p*(h(x)-u(x)))-0.5)^2` with `c` being a hyperparameter and `p` learned.
      * `epsilon`: Either `z` or `|z|`. If `z` is a Gaussian, then `|z|` is called "half-normal" while just `z` is called "normal". Half-normal lets the noise only point towards one "direction" (towards the unsaturated regime or away from it), while normal noise lets it point in both directions (with the slope being influenced by `alpha`).
    * The formula can be split into three parts:
      * `alpha*h(x)`: Nonlinear part.
      * `(1-alpha)u(x)`: Linear part.
      * `d(x)std(x)epsilon`: Stochastic part.
    * Each of these parts resembles a path along which gradient can flow through the network.
    * During test time the activation function is made deterministic by using its expectation value: `E[phi(x,z)] = alpha*h(x) + (1-alpha)u(x) + d(x)std(x)E[epsilon]`.
    * If `z` is half-normal then `E[epsilon] = sqrt(2/pi)`. If `z` is normal then `E[epsilon] = 0`.

* (5) Adding Noise to Input of the Function
  * Noise can also be added to the input of an activation function, i.e. `h(x)` becomes `h(x + noise)`.
  * The noise can either always be applied or only once the input passes a threshold.

* (6) Experimental Results
  * They applied noise only during training.
  * They used existing setups and just changed the activation functions to noisy ones. No further optimizations.
  * `p` was initialized uniformly to [-1,1].
  * Basic experiment settings:
    * NAN: Normal noise applied to the outputs.
    * NAH: Half-normal noise, i.e. `|z|`, i.e. noise is "directed" towards the unsaturated or satured regime.
    * NANI: Normal noise applied to the *input*, i.e. `h(x+noise)`.
    * NANIL: Normal noise applied to the input with learned variance.
    * NANIS: Normal noise applied to the input, but only if the unit saturates (i.e. above/below thresholds).
  * (6.1) Exploratory analysis
    * A very simple MNIST network performed slightly better with noisy activations than without. But comparison was only to tanh and hard-tanh, not ReLU or similar.
    * In an experiment with a simple GRU, NANI (noisy input) and NAN (noisy output) performed practically identical. NANIS (noisy input, only when saturated) performed significantly worse.
  * (6.2) Learning to Execute
    * Problem setting: Predict the output of some lines of code.
    * They replaced sigmoids and tanhs with their noisy counterparts (NAH, i.e. half-normal noise on output). The model learned faster.
  * (6.3) Penntreebank Experiments
    * They trained a standard 2-layer LSTM language model on Penntreebank.
    * Their model used noisy activations, as opposed to the usually non-noisy ones.
    * They could improve upon the previously best value. Normal noise and half-normal noise performed roughly the same.
  * (6.4) Neural Machine Translation Experiments
    * They replaced all sigmoids and tanh units in the Neural Attention Model with noisy ones. Then they trained on the Europarl corpus.
    * They improved upon the previously best score.
  * (6.5) Image Caption Generation Experiments
    * They train a network with soft attention to generate captions for the Flickr8k dataset.
    * Using noisy activation units improved the result over normal sigmoids and tanhs.
  * (6.6) Experiments with Continuation
    * They build an LSTM and train it to predict how many unique integers there are in a sequence of random integers.
    * Instead of using a constant value for hyperparameter `c` of the noisy activations (scale of the standard deviation of the noise), they start at `c=30` and anneal down to `c=0.5`.
    * Annealed noise performed significantly better then unannealed noise.
    * Noise applied to the output (NAN) significantly beat noise applied to the input (NANIL).
    * In a second experiment they trained a Neural Turing Machine on the associative recall task.
    * Again they used annealed noise.
    * The NTM with annealed noise learned by far faster than the one without annealed noise and converged to a perfect solution.
