# Paper

* **Title**: InfoGAN: Interpretable Representation Learning by Information Maximizing Generative Adversarial Nets
* **Authors**: Xi Chen, Yan Duan, Rein Houthooft, John Schulman, Ilya Sutskever, Pieter Abbeel
* **Link**: https://arxiv.org/abs/1606.03657
* **Tags**: Neural Network, GAN
* **Year**: 2016

# Summary

* What
  * Usually GANs transform a noise vector `z` into images. `z` might be sampled from a normal or uniform distribution.
  * The effect of this is, that the components in `z` are deeply entangled.
    * Changing single components has hardly any influence on the generated images. One has to change multiple components to affect the image.
    * The components end up not being interpretable. Ideally one would like to have meaningful components, e.g. for human faces one that controls the hair length and a categorical one that controls the eye color.
  * They suggest a change to GANs based on Mutual Information, which leads to interpretable components.
    * E.g. for MNIST a component that controls the stroke thickness and a categorical component that controls the digit identity (1, 2, 3, ...).
    * These components are learned in a (mostly) unsupervised fashion.

* How
  * The latent code `c`
    * "Normal" GANs parameterize the generator as `G(z)`, i.e. G receives a noise vector and transforms it into an image.
    * This is changed to `G(z, c)`, i.e. G now receives a noise vector `z` and a latent code `c` and transforms both into an image.
    * `c` can contain multiple variables following different distributions, e.g. in MNIST a categorical variable for the digit identity and a gaussian one for the stroke thickness.
  * Mutual Information
    * If using a latent code via `G(z, c)`, nothing forces the generator to actually use `c`. It can easily ignore it and just deteriorate to `G(z)`.
    * To prevent that, they force G to generate images `x` in a way that `c` must be recoverable. So, if you have an image `x` you must be able to reliable tell which latent code `c` it has, which means that G must use `c` in a meaningful way.
    * This relationship can be expressed with mutual information, i.e. the mutual information between `x` and `c` must be high.
      * The mutual information between two variables X and Y is defined as `I(X; Y) = entropy(X) - entropy(X|Y) = entropy(Y) - entropy(Y|X)`.
      * If the mutual information between X and Y is high, then knowing Y helps you to decently predict the value of X (and the other way round).
      * If the mutual information between X and Y is low, then knowing Y doesn't tell you much about the value of X (and the other way round).
    * The new GAN loss becomes `old loss - lambda * I(G(z, c); c)`, i.e. the higher the mutual information, the lower the result of the loss function.
  * Variational Mutual Information Maximization
    * In order to minimize `I(G(z, c); c)`, one has to know the distribution `P(c|x)` (from image to latent code), which however is unknown.
    * So instead they create `Q(c|x)`, which is an approximation of `P(c|x)`.
    * `I(G(z, c); c)` is then computed using a lower bound maximization, similar to the one in variational autoencoders (called "Variational Information Maximization", hence the name "InfoGAN").
    * Basic equation: `LowerBoundOfMutualInformation(G, Q) = E[log Q(c|x)] + H(c) <= I(G(z, c); c)`
      * `c` is the latent code.
      * `x` is the generated image.
      * `H(c)` is the entropy of the latent codes (constant throughout the optimization).
      * Optimization w.r.t. Q is done directly.
      * Optimization w.r.t. G is done via the reparameterization trick.
    * If `Q(c|x)` approximates `P(c|x)` *perfectly*, the lower bound becomes the mutual information ("the lower bound becomes tight").
    * In practice, `Q(c|x)` is implemented as a neural network. Both Q and D have to process the generated images, which means that they can share many convolutional layers, significantly reducing the extra cost of training Q.

* Results
  * MNIST
    * They use for `c` one categorical variable (10 values) and two continuous ones (uniform between -1 and +1).
    * InfoGAN learns to associate the categorical one with the digit identity and the continuous ones with rotation and width.
    * Applying Q(c|x) to an image and then classifying only on the categorical variable (i.e. fully unsupervised) yields 95% accuracy.
    * Sampling new images with exaggerated continuous variables in the range `[-2,+2]` yields sound images (i.e. the network generalizes well).
    * ![MNIST examples](images/InfoGAN__mnist.png?raw=true "MNIST examples")
  * 3D face images
    * InfoGAN learns to represent the faces via pose, elevation, lighting.
    * They used five uniform variables for `c`. (So two of them apparently weren't associated with anything sensible? They are not mentioned.)
  * 3D chair images
    * InfoGAN learns to represent the chairs via identity (categorical) and rotation or width (apparently they did two experiments).
    * They used one categorical variable (four values) and one continuous variable (uniform `[-1, +1]`).
  * SVHN
    * InfoGAN learns to represent lighting and to spot the center digit.
    * They used four categorical variables (10 values each) and two continuous variables (uniform `[-1, +1]`). (Again, a few variables were apparently not associated with anything sensible?)
  * CelebA
    * InfoGAN learns to represent pose, presence of sunglasses (not perfectly), hair style and emotion (in the sense of "smiling or not smiling").
    * They used 10 categorical variables (10 values each). (Again, a few variables were apparently not associated with anything sensible?)
    * ![CelebA examples](images/InfoGAN__celeba.png?raw=true "CelebA examples")
