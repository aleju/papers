# Paper

* **Title**: Precomputed Real-Time Texture Synthesis with Markovian Generative Adversarial Networks
* **Authors**: Chuan Li, Michael Wand
* **Link**: http://arxiv.org/abs/1604.04382
* **Tags**: Neural Network, MRF, GAN, generative, artistic style
* **Year**: 2016

# Summary

* See also
  * [Video explanation by authors](https://www.youtube.com/watch?v=PRD8LpPvdHI)

* What
  * They describe a method that can be used for two problems:
    * (1) Choose a style image and apply that style to other images.
    * (2) Choose an example texture image and create new texture images that look similar.
  * In contrast to previous methods their method can be applied very fast to images (style transfer) or noise (texture creation). However, per style/texture a single (expensive) initial training session is still necessary.
  * Their method builds upon their previous paper "[Combining Markov Random Fields and Convolutional Neural Networks for Image Synthesis](Combining_MRFs_and_CNNs_for_Image_Synthesis.md)".

* How
  * Rough overview of their previous method:
    * Transfer styles using three losses:
      * Content loss: MSE between VGG representations.
      * Regularization loss: Sum of x-gradient and y-gradients (encouraging smooth areas).
      * MRF-based style loss: Sample `k x k` patches from VGG representations of content image and style image. For each patch from content image find the nearest neighbor (based on normalized cross correlation) from style patches. Loss is then the sum of squared errors of euclidean distances between content patches and their nearest neighbors.
    * Generation of new images is done by starting with noise and then iteratively applying changes that minimize the loss function.
  * They introduce mostly two major changes:
    * Get rid of the costly nearest neighbor search for the MRF loss. Instead, use a discriminator-network that receives a patch and rates how real that patch looks.
      * This discriminator-network is costly to train, but that only has to be done once (per style/texture).
    * Get rid of the slow, iterative generation of images. Instead, start with the content image (style transfer) or noise image (texture generation) and feed that through a single generator-network to create the output image (with transfered style or generated texture).
      * This generator-network is costly to train, but that only has to be done once (per style/texture). 
  * MDANs:
  * MGANs:

* Results

![Examples](images/Accurate_Image_Super-Resolution__examples.png?raw=true "Examples")

*Super-resolution quality of their model (top, bottom is a competing model).*
