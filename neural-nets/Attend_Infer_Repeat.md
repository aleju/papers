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
    * Inference for their model is intractable, therefore they use an approximation `q(z,n|x)`, which minizes `KL(q(z,n|x)||p(z,n|x))`, i.e. KL(approximation||real) using amortized variational approximation.
    * Challanges for them:
      * The dimensionality of their latent variable layer is a random variable p(n) (i.e. No static size.).
      * Strong symmetries.
    * They implement inference via an RNN which encodes the image object by object.
    * The encoded latent variables can be gaussians.
    * They encode the latent layer length via n as a vector (instead of an integer). The vector has the form of n 1s followed by one 0.
    * If the length vector is `#z` then they want to approximate `q(z,#z|x)`.
    * That can apparently be decomposed into `<product> q(latent variable value i, #z is still 1 at i|x, previous latent variable values) * q(has length n|z,x)`.
  * (2.2) Learning
    * The parameters theta (`p`, latent variable -> image) and phi (`q`, image -> latent variables) are jointly optimized.
    * Optimization happens be maximizing a lower bound `E[log(p(x,z,n) / q(z,n|x))]` called the negative free energy.
    * (2.2.1) Parameters of the model theta
      * 
