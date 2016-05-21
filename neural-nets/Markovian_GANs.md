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
    * (a) Get rid of the costly nearest neighbor search for the MRF loss. Instead, use a discriminator-network that receives a patch and rates how real that patch looks.
      * This discriminator-network is costly to train, but that only has to be done once (per style/texture).
    * (b) Get rid of the slow, iterative generation of images. Instead, start with the content image (style transfer) or noise image (texture generation) and feed that through a single generator-network to create the output image (with transfered style or generated texture).
      * This generator-network is costly to train, but that only has to be done once (per style/texture). 
  * MDANs
    * They implement change (a) to the standard architecture and call that an "MDAN" (Markovian Deconvolutional Adversarial Networks).
    * So the architecture of the MDAN is:
      * Input: Image (RGB pixels)
      * Branch 1: Markovian Patch Quality Rater (aka Discriminator)
        * Starts by feeding the image through VGG19 until layer `relu3_1`. (Note: VGG weights are fixed/not trained.)
        * Then extracts `k x k` patches from the generated representations.
        * Feeds each patch through a shallow ConvNet (convolution with BN then fully connected layer).
        * Training loss is a hinge loss, i.e. max margin between classes +1 (real looking patch) and -1 (fake looking patch). (Could also take a single sigmoid output, but they argue that hinge loss isn't as likely to saturate.)
        * This branch will be trained continuously while synthesizing a new image.
      * Branch 2: Content Estimation/Guidance
        * Note: This branch is only used for style transfer, i.e if using an content image and not for texture generation.
        * Starts by feeding the currently synthesized image through VGG19 until layer `relu5_1`. (Note: VGG weights are fixed/not trained.)
        * Also feeds the content image through VGG19 until layer `relu5_1`.
        * Then uses a MSE loss between both representations (so similar to a MSE on RGB pixels that is often used in autoencoders).
        * Nothing in this branch needs to trained, the loss only affects the synthesizing of the image.
  * MGANs
    * The MGAN is like the MDAN, but additionally implements change (b), i.e. they add a generator that takes an image and stylizes it.
    * The generator's architecture is:
      * Input: Image (RGB pixels) or noise (for texture synthesis)
      * Output: Image (RGB pixels) (stylized input image or generated texture)
      * The generator takes the image (pixels) and feeds that through VGG19 until layer `relu4_1`.
      * Similar to the DCGAN generator, they then apply a few fractionally strided convolutions (with BN and LeakyReLUs) to that, ending in a Tanh output. (Fractionally strided convolutions increase the height/width of the images, here to compensate the VGG pooling layers.)
      * The output after the Tanh is the output image (RGB pixels).
    * They train the generator with pairs of `(input image, stylized image or texture)`. These pairs can be gathered by first running the MDAN alone on several images. (With significant augmentation a few dozen pairs already seem to be enough.)
    * One of two possible loss functions can then be used:
      * Simple standard choice: MSE on the euclidean distance between expected output pixels and generated output pixels. Can cause blurriness.
      * Better choice: MSE on a higher VGG representation. Simply feed the generated output pixels through VGG19 until `relu4_1` and the reuse the already generated (see above) VGG-representation of the input image. This is very similar to the pixel-wise comparison, but tends to cause less blurriness.
    * Note: For some reason the authors call their generator a VAE, but don't mention any typical VAE technique, so it's not described like one here.
  * They use Adam to train their networks.
  * For texture generation they use Perlin Noise instead of simple white noise. In Perlin Noise, lower frequency components dominate more than higher frequency components. White noise didn't work well with the VGG representations in the generator (activations were close to zero).

* Results
  * Similar quality like previous methods, but much faster (compared to most methods).
  * For the Markovian Patch Quality Rater (MDAN branch 1):
    * They found that the weights of this branch can be used as initialization for other training sessions (e.g. other texture styles), leading to a decrease in required iterations/epochs.
    * Using VGG for feature extraction seems to be crucial. Training from scratch generated in worse results.
    * Using larger patch sizes preserves more structure of the structure of the style image/texture. Smaller patches leads to more flexibility in generated patterns.
    * They found that using more than 3 convolutional layers or more than 64 filters per layer provided no visible benefit in quality.


![Example](images/Markovian_GANs__example.png?raw=true "Example")

*Result of their method, compared to other methods.*


![Architecture](images/Markovian_GANs__architecture.png?raw=true "Architecture")

*Architecture of their model.*
