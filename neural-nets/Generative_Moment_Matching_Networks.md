# Paper

* **Title**: Generative Moment Matching Networks
* **Authors**: Yujia Li, Kevin Swersky, Richard Zemel
* **Link**: http://arxiv.org/abs/1502.02761
* **Tags**: Neural Network, generative model
* **Year**: 2015

# Summary


--------------------

# Rough chapter-wise notes

* (1) Introduction
  * Sampling in GMMNs is fast.
  * GMMNs are similar to GANs.
  * While the training objective in GANs is a minimax problem, in GMMNs it is a simple loss function.
  * GMMNs are based on maximum mean discrepancy. They use that (implemented via the kernel trick) as the loss function.
  * GMMNs try to generate data so that the moments in the generated data are as similar as possible to the moments in the training data.
  * They combine GMMNs with autoencoders. That is, they first train an autoencoder to generate images. Then they train a GMMN to produce sound code inputs to the decoder of the autoencoder.

* (2) Maximum Mean Discrepancy
  * Maximum mean discrepancy (MMD) is a frequentist estimator to tell whether two datasets X and Y come from the same probability distribution.
  * MMD estimates basic statistics values (i.e. mean and higher order statistics) of both datasets and compares them with each other.
  * MMD can be formulated so that examples from the datasets are only used for scalar products. Then the kernel trick can be applied.
  * It can be shown that minimizing MMD with gaussian kernels is equivalent to matching all moments between the probability distributions of the datasets.

* (4) Generative Moment Matching Networks
  * Data Space Networks
    * Just like GANs, GMMNs start with a noise vector that has N values sampled uniformly from [-1, 1].
    * The noise vector is then fed forward through several fully connected ReLU layers.
    * The MMD is differentiable and therefor can be used for backpropagation.
  * Auto-Encoder Code Sparse Networks
    * AEs can be used to reconstruct high-dimensional data, which is a simpler task than to learn to generate new data from scratch.
    * Advantages of using the AE code space:
      * Dimensionality can be explicitly chosen.
      * Disentangling factors of variation.
    * 
