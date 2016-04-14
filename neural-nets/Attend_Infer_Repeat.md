# Paper

* **Title**: Attend, Infer, Repeat: Fast Scene Understanding with Generative Models
* **Authors**: S. M. Ali Eslami, Nicolas Heess, Theophane Weber, Yuval Tassa, Koray Kavukcuoglu, Geoffrey E. Hinton
* **Link**: http://arxiv.org/abs/1603.08575
* **Tags**: Neural Network, VAE, attention
* **Year**: 2016

# Summary

![Architecture](images/DenseCap__architecture.png?raw=true "Architecture.")



--------------------

# Rough chapter-wise notes

* (1) Introduction
  * Assumption: Images are made up of distinct objects. These objects have visual and physical properties.
  * They develop a framework for efficient inference in images (i.e. get from the image to a representation of the objects, i.e. inverse graphics).
  * Parts of the framework: High dimensional representations (e.g. object images), interpretable latent variables (e.g. for rotation) and generative processes (to combine object images with latent variables).
  * Contributions:
    * A scheme for efficient variational inference in latent spaces of variable dimensionality.
      * Idea: Treat inference as an iterative process, implemented via an RNN that looks at one object at a time and learns an appropriate number of inference steps. (Attent-Infer-Repeat, AIR)
      * End-to-end training via amortized variational inference (continuous variables: gradient descent, discrete variables: black-box optimization).
    * AIR allows to train generative models that automatically learn to decompose scenes.
    * AIR allows to recover objects and their attributes from rendered 3D scenes (inverse rendering).

* (2) Approach
  * Just like in VAEs, the scene interpretation is viewed with a bayesdian approach.
  * There are latent variables `z` and images `x`.
  * Images are generated via a probability distribution `p(x|z)`.
  * This can be reversed via bayes rule to `p(x|z) = p(x)p(z|x) / p(z)` which means that `p(x|z)p(z) / p(x) = p(z|x)`.
  * The prior `p(z)` must be chosen and captures assumption about the distributions of the latent variables.
  * `p(x|z)` is the likelihood and represents the model that generates images from latent variables.
  * They assume that there can be multiple objects in an image.
  * Every object get its own latent variables.
  * A probability distribution p(x|z) then converts each object (on its own) from the latent variables to an image.
  * The number of objects follows a probability distribution `p(n)`.
  * For the prior and likelihood they assume two scenarios:
    * 2D: Three dimensions for X, Y and scale. Additionally n dimensions for its shape.
    * 3D: Dimensions for X, Y, Z, rotation, object identity/category (multinomial variable). (No scale?)
  * Both 2D and 3D can be separated into latent variables for "where" and "what".
  * It is assumed that the prior latent variables are independent of each other.
  * (2.1) Inference

