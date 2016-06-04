# Paper

* **Title**: Generating images with recurrent adversarial networks
* **Authors**: Daniel Jiwoong Im, Chris Dongjoo Kim, Hui Jiang, Roland Memisevic
* **Link**: http://arxiv.org/abs/1602.05110v4
* **Tags**: Neural Network, GAN, residual
* **Year**: 2016

# Summary

* What
  * They describe a new architecture for GANs.
  * The architecture is based on letting the Generator (G) create images in multiple steps, similar to DRAW.
  * They also briefly suggest a method to compare the quality of the results of different generators with each other.

* How
  * In a classic GAN one samples a noise vector `z`, feeds that into a Generator (`G`), which then generates an image `x`, which is then fed through the Discriminator (`D`) to estimate its quality.
  * Their method operates in basically the same way, but internally G is changed to generate images in multiple time steps.
  * Outline of how their G operates:
    * Time step 0:
      * Input: Empty image `delta C-1`, randomly sampled `z`.
      * Feed `delta C-1` through a number of downsampling convolutions to create a tensor. (Not very useful here, as the image is empty. More useful in later timesteps.)
      * Feed `z` through a number of upsampling convolutions to create a tensor (similar to DCGAN).
      * Concat the output of the previous two steps.
      * Feed that concatenation through a few more convolutions.
      * Output: `delta C0` (changes to apply to the empty starting canvas).
    * Time step 1 (and later):
      * Input: Previous change `delta C0`, randomly sampled `z` (can be the same as in step 0).
      * Feed `delta C0` through a number of downsampling convolutions to create a tensor.
      * Feed `z` through a number of upsampling convolutions to create a tensor (similar to DCGAN).
      * Concat the output of the previous two steps.
      * Feed that concatenation through a few more convolutions.
      * Output: `delta C1` (changes to apply to the empty starting canvas).
    * At the end, after all timesteps have been performed:
      * Create final output image by summing all the changes, i.e. `delta C0 + delta C1 + ...`, which basically means `empty start canvas + changes from time step 0 + changes from time step 1 + ...`.
  * Their architecture as an image:
    * ![Architecture](images/Generating_Images_with_Recurrent_Adversarial_Networks__architecture.png?raw=true "Architecture")
  * Comparison measure
    * They suggest a new method to compare GAN results with each other.
    * They suggest to train pairs of G and D, e.g. for two pairs (G1, D1), (G2, D2). Then they let the pairs compete with each other.
    * To estimate the quality of D they suggest `r_test = errorRate(D1, testset) / errorRate(D2, testset)`. ("Which D is better at spotting that the test set images are real images?")
    * To estimate the quality of the generated samples they suggest `r_sample = errorRate(D1, images by G2) / errorRate(D2, images by G1)`. ("Which G is better at fooling an unknown D, i.e. possibly better at generating life-like images?")
    * They suggest to estimate which G is better using r_sample and then to estimate how valid that result is using r_test.

* Results
  * Generated images of churches, with timesteps 1 to 5:
    * ![Churches](images/Generating_Images_with_Recurrent_Adversarial_Networks__churches.jpg?raw=true "Churches")
  * Overfitting
    * They saw no indication of overfitting in the sense of memorizing images from the training dataset.
    * They however saw some indication of G just interpolating between some good images and of G reusing small image patches in different images.
  * Randomness of noise vector `z`:
    * Sampling the noise vector once seems to be better than resampling it at every timestep.
    * Resampling it at every time step often led to very similar looking output images.
