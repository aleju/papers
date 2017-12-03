# Paper

* **Title**: High-Resolution Image Synthesis and Semantic Manipulation with Conditional GANs
* **Authors**: Ting-Chun Wang, Ming-Yu Liu, Jun-Yan Zhu, Andrew Tao, Jan Kautz, Bryan Catanzaro
* **Link**: https://arxiv.org/abs/1711.11585
* **Tags**: Neural Network, GAN, segmentation
* **Year**: 2017

# Other

* [website](https://tcwang0509.github.io/pix2pixHD/)
* [video](https://www.youtube.com/watch?v=3AIpPlzM_qs)

# Summary

* What
  * They suggest a method to generate (via GANs) high-resolution images that match given segmentation maps.
  * They introduce a new loss for GANs.
  * They suggest a method to easily change the appearances of object instances in the images.

* How
  * Architecture
    * They train two generators (theoretically can be any number of generators).
    * The first (small-scale) generator gets a small-scale segmentation map as its input (e.g. a quarter of the intended output size).
    * It generates an image matching that segmentation map -- as in any other GAN.
    * The small scale generator is then trained for some time.
    * Then they add a second generator (large-scale).
    * At each execution, first the small-scale generator is executed. It receives the small-scale segmentation map as input, transforms that into a feature map and uses that map to generate an output image.
    * Then the large-scale generator is executed. It first uses convolutions to downscale the large-scale segmentation map. Then it adds (in residual fashion) the feature map from the small-scale generator. Then it uses these features to generate the large-scale output image.
    * Visualization:
      * ![architecture](images/High_Resolution_Image_Synthesis_with_Conditional_GANs/architecture.jpg?raw=true "architecture")
    * Additionally, they train three different discriminators.
    * Each discriminator works completely independently and gets a different scale as input (e.g. D1: 1x downscaled, D2: 2x downscaled, D3: 4x downscaled).
    * Each image produced by the generator is fed through all three discriminators.
    * This allows to do multi-scale discrimination (training the generator at the coarse and fine level).
  * Loss
    * They add a feature matching loss, which incentivizes the generator to produce images that are projected by the discriminator onto features that are similar to real images.
    * Equation:
      * ![feature matching](images/High_Resolution_Image_Synthesis_with_Conditional_GANs/feature_matching.jpg?raw=true "feature matching")
    * They add a perceptual loss, which works similar to the feature matching loss, but the network to generate the features is VGG16 (instead of the discriminator).
    * Equation:
      * ![perceptual loss](images/High_Resolution_Image_Synthesis_with_Conditional_GANs/perceptual_loss.jpg?raw=true "perceptual loss")
    * They use LSGAN (least squares GAN) for everything else.
  * Instance Segmentation
    * The generator gets a semantic segmentation map as its input.
    * Downside: The map does not indicate where each object instance ends. This makes the image generation needlessly more complicated.
    * To fix that, they add instance information.
    * Just adding the instance segmentation map is difficult, as there can be infinitely many instances, resulting in an input with infinitely many channels.
    * Instead, they just use the segmentation map as one channel and add the *boundaries* of the object instances as a second channel.
    * Visualization:
      * ![boundary maps](images/High_Resolution_Image_Synthesis_with_Conditional_GANs/boundary_maps.jpg?raw=true "boundary maps")
  * Instance Manipulation
    * Their aim is to develop a tool in which one can select an object instance and easily change its appearance.
    * That appearance change could be created by changing the generator's input noise vector.
    * Downside 1: This would also change everything else in the image.
    * Downside 2: Potentially many noise vectors might result in practically the same appearance. So many vectors would have to be tried by the user.
    * They use an instance-wise feature embedding to fix that.
    * Step 1:
      * Use an autoencoder-like network `E` to transform each input image into a feature map.
      * `E` has an encoder that encodes the image into a small (bottleneck) feature map.
      * `E` has e decoder that decodes the bottleneck into a large feature map (not an image!) of the same size as the input image.
      * `E`'s output feature map could be added to the generator's input, i.e. no special autoencoder loss is needed.
    * Step 2:
      * `E` has transformed the input image to a feature map.
      * Apply average pooling to that feature map *per object instance*.
      * This effectively creates one feature vector per object instance (upscaled so that it cover's the object instance's area in the original image). The feature vector contains information about the object instance's appearance.
      * They set each feature vector to have three components (i.e. `E`'s output has three channels).
      * Visualization:
        * ![feature encoder network](images/High_Resolution_Image_Synthesis_with_Conditional_GANs/feature_encoder.jpg?raw=true "feature encoder network")
    * Step 3:
      * Now train the generator as before, but additionally to the semantic segmentation maps it gets the pooled feature maps as inputs.
      * As mentioned, `E` is trained in parallel.
    * Step 4:
      * Generator and `E` are now trained.
      * Convert each image in the dataset to a feature map via `E`. This results in a large number of instance-specific appearance vectors.
      * Run K-Means clustering on these vectors with K=10 clusters per class.
      * Result: Per object class 10 different appearance vectors. By changing the vector of on object instance to a different one (among these 10), the object's appearance in the image can be changed without significant effects on the remainder of the image.

* Results
  * Subjectively more realistic looking images than pix2pix and CRF at 2048x1024.
  * Tests with Mechanical Turk also show that their images look more realistic to most people.
  * Tests with Mechanical Turk indicate that both feature loss and perceptual loss (based on VGG16) improve image quality. (Though numbers for perceptual loss aren't that clear.)
  * Tests show that the generated images have high matching with the segmentation maps given to the generator.
  * Adding instance boundaries as an input channel helps with image quality at object boundaries:
    * ![with boundary maps](images/High_Resolution_Image_Synthesis_with_Conditional_GANs/with_boundary_maps.jpg?raw=true "with boundary maps")
  * Easy changing of object instance appearances, see video.
  * They also test on other datasets (NYU, ADE20K, Helen Faces) and get good results there too.

