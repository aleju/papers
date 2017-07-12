# Paper

* **Title**: Self-Normalizing Neural Networks
* **Authors**: Günter Klambauer, Thomas Unterthiner, Andreas Mayr, Sepp Hochreiter
* **Link**: https://arxiv.org/abs/1706.02515
* **Tags**: Neural Network, Normalization, ELU
* **Year**: 2017

# See also

* [paper's github repository](https://github.com/bioinf-jku/SNNs)

# Summary

* What
  * They suggest a variation of ELUs, which leads to networks being automatically normalized.
  * The effects are comparable to Batch Normalization, while requiring significantly less computation (barely more than a normal ReLU).

* How
  * They define Self-Normalizing Neural Networks (SNNs) as neural networks, which automatically keep their activations at zero-mean and unit-variance (per neuron).
  * SELUs
    * They use SELUs to turn their networks into SNNs.
    * Formula:
      * ![SELU](images/Self-Normalizing_Neural_Networks__SELU.jpg?raw=true "SELU")
      * with `alpha = 1.6733` and `lambda = 1.0507`.
    * They proof that with properly normalized weights the activations approach a fixed point of zero-mean and unit-variance. (Different settings for alpha and lambda can lead to other fixed points.)
    * They proof that this is still the case when previous layer activations and weights do not have optimal values.
    * They proof that this is still the case when the variance of previous layer activations is very high or very low and argue that the mean of those activations is not so important.
    * Hence, SELUs with these hyperparameters should have self-normalizing properties.
    * SELUs are here used as a basis because:
      1. They can have negative and positive values, which allows to control the mean.
      2. They have saturating regions, which allows to dampen high variances from previous layers.
      3. They have a slope larger than one, which allows to increase low variances from previous layers.
      4. They generate a continuous curve, which ensures that there is a fixed point between variance damping and increasing.
    * ReLUs, Leaky ReLUs, Sigmoids and Tanhs do not offer the above properties.
  * Initialization
    * SELUs for SNNs work best with normalized weights.
    * They suggest to make sure per layer that:
      1. The first moment (sum of weights) is zero.
      2. The second moment (sum of squared weights) is one.
    * This can be done by drawing weights from a normal distribution `N(0, 1/n)`, where `n` is the number of neurons in the layer.
  * Alpha-dropout
    * SELUs don't perform as well with normal Dropout, because their point of low variance is not 0.
    * They suggest a modification of Dropout called Alpha-dropout.
    * In this technique, values are not dropped to 0 but to `alpha' = -lambda * alpha = -1.0507 * 1.6733 = -1.7581`.
    * Similar to dropout, activations are changed during training to compensate for the dropped units.
    * Each activation `x` is changed to `a(xd+alpha'(1-d))+b`.
      * `d = B(1, q)` is the dropout variable consisting of 1s and 0s.
      * `a = (q + alpha'^2 q(1-q))^(-1/2)`
      * `b = -(q + alpha'^2 q(1-q))^(-1/2) ((1-q)alpha')`
    * They made good experiences with dropout rates around 0.05 to 0.1.

* Results
  * Note: All of their tests are with fully connected networks. No convolutions.
  * Example training results:
    * ![MINST CIFAR10](images/Self-Normalizing_Neural_Networks__MNIST_CIFAR10.jpg?raw=true "MNIST CIFAR10")
    * Left: MNIST, Right: CIFAR10
    * Networks have N layers each, see legend. No convolutions.
  * 121 UCI Tasks
    * They manage to beat SVMs and RandomForests, while other networks (Layer Normalization, BN, Weight Normalization, Highway Networks, ResNet) perform significantly worse than their network (and usually don't beat SVMs/RFs).
  * Tox21
    * They achieve better results than other networks (again, Layer Normalization, BN, etc.).
    * They achive almost the same result as the so far best model on the dataset, which consists of a mixture of neural networks, SVMs and Random Forests.
  * HTRU2
    * They achieve better results than other networks.
    * They beat the best non-neural method (Naive Bayes).
  * Among all tested other networks, MSRAinit performs best, which references a network withput any normalization, only ReLUs and Microsoft Weight Initialization (see paper: `Delving deep into rectifiers: Surpassing human-level performance on imagenet classification`).

