# Paper

* **Title**: StackGAN: Text to Photo-realistic Image Synthesis with Stacked Generative Adversarial Networks
* **Authors**: Han Zhang, Tao Xu, Hongsheng Li, Shaoting Zhang, Xiaolei Huang, Xiaogang Wang, Dimitris Metaxas
* **Link**: https://arxiv.org/abs/1612.03242
* **Tags**: Neural Network, GAN
* **Year**: 2016

# Summary

* What
  * They propose a two-stage GAN architecture that generates 256x256 images of (relatively) high quality.
  * The model gets text as an additional input and the images match the text.

* How
  * Most of the architecture is the same as in any GAN:
    * Generator G generates images.
    * Discriminator D discriminates betweens fake and real images.
    * G gets a noise variable `z`, so that it doesn't always do the same thing.
  * Two-staged image generation:
    * Instead of one step, as in most GANs, they use two steps, each consisting of a G and D.
    * The first generator creates 64x64 images via upsampling.
    * The first discriminator judges these images via downsampling convolutions.
    * The second generator takes the image from the first generator, downsamples it via convolutions, then applies some residual convolutions and then re-upsamples it to 256x256.
    * The second discriminator is comparable to the first one (downsampling convolutions).
    * Note that the second generator does not get an additional noise term `z`, only the first one gets it.
    * For upsampling, they use 3x3 convolutions with ReLUs, BN and nearest neighbour upsampling.
    * For downsampling, they use 4x4 convolutions with stride 2, Leaky ReLUs and BN (the first convolution doesn't seem to use BN).
  * Text embedding:
    * The generated images are supposed to match input texts.
    * These input texts are embedded to vectors.
    * These vectors are added as:
      1. An additional input to the first generator.
      2. An additional input to the second generator (concatenated after the downsampling and before the residual convolutions).
      3. An additional input to the first discriminator (concatenated after the downsampling).
      4. An additional input to the second discriminator (concatenated after the downsampling).
    * In case the text embeddings need to be matrices, the values are simply reshaped to `(N, 1, 1)` and then repeated to `(N, H, W)`.
    * The texts are converted to embeddings via a network at the start of the model.
      * Input to that vector: Unclear. (Concatenated word vectors? Seems to not be described in the text.)
      * The input is transformed to a vector via a fully connected layer (the text model is apparently not recurrent).
      * The vector is transformed via fully connected layers to a mean vector and a sigma vector.
      * These are then interpreted as normal distributions, from which the final output vector is sampled. This uses the reparameterization trick, similar to the method in VAEs.
      * Just like in VAEs, a KL-divergence term is added to the loss, which prevents each single normal distribution from deviating too far from the unit normal distribution `N(0,1)`.
      * The authors argue, that using the VAE-like formulation -- instead of directly predicting an output vector (via FC layers) -- compensated for the lack of labels (smoother manifold).
      * Note: This way of generating text embeddings seems very simple. (No recurrence, only about two layers.) It probably won't do much more than just roughly checking for the existence of specific words and word combinations (e.g. "red head").
  * Visualization of the architecture:
    * ![Architecture](images/StackGAN__architecture.jpg?raw=true "Architecture")

* Results
  * Note: No example images of the two-stage architecture for LSUN bedrooms.
  * Using only the first stage of the architecture (first G and D) reduces the Inception score significantly.
  * Adding the text to both the first and second generator improves the Inception score slightly.
  * Adding the VAE-like text embedding generation (as opposed to only FC layers) improves the Inception score slightly.
  * Generating images at higher resolution (256x256 instead of 128x128) improves the Inception score significantly
    * Note: The 256x256 architecture has more residual convolutions than the 128x128 one.
    * Note: The 128x128 and the 256x256 are both upscaled to 299x299 images before computing the Inception score. That should make the 128x128 images quite blurry and hence of low quality.
  * Example images, with text and stage 1/2 results:
    * ![Examples](images/StackGAN__examples.jpg?raw=true "Examples")
  * More examples of birds:
    * ![Examples birds](images/StackGAN__examples_birds.jpg?raw=true "Examples birds")
  * Examples of failures:
    * ![Failure Cases](images/StackGAN__failures.jpg?raw=true "Failure Cases")
    * The authors argue, that most failure cases happen when stage 1 messes up.

