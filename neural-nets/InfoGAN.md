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

* Results
  * 

![Architecture](images/Accurate_Image_Super-Resolution__architecture.png?raw=true "Architecture")

*Architecture of the model.*
