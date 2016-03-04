# Paper

* **Title**: Deep Generative Image Models using a Laplacian Pyramid of Adversarial Networks
* **Authors**: Emily Denton, Soumith Chintala, Arthur Szlam, Rob Fergus
* **Link**: http://arxiv.org/abs/1506.05751
* **Tags**: Neural Network, GAN, generative models, Laplacian Pyramid
* **Year**: 2015

# Summary



![Activations](images/Deep_Residual_Learning_for_Image_Recognition__activations.png?raw=true "Activations")



-------------------------

# Rough chapter-wise notes

* Introduction
  * Instead of just one big generative model, they build multiple ones.
  * They start with one model at a small image scale (e.g. 4x4) and then add multiple generative models that increase the image size (e.g. from 4x4 to 8x8).
  * This scaling from coarse to fine (low frequency to high frequency components) resembles a laplacian pyramid, hence the name of the paper.

* Related Works
  * Types of generative image models:
    * Non-Parametric: Models copy patches from training set (e.g. texture synthesis, super-resolution)
    * Parametric: E.g. Deep Boltzmann machines or denoising auto-encoders
  * Novel approaches: e.g. DRAW, diffusion-based processes, LSTMs
  * This work is based on (conditional) GANs

* Approach
  * They start with a Gaussian and a Laplacian pyramid.
  * They build the Gaussian pyramid by repeatedly decreasing the image height/width by 2: [full size image, half size image, quarter size image, ...]
  * They build a Laplacian pyramid by taking pairs of images in the gaussian pyramid, upscaling the smaller one and then taking the difference.
  * 
