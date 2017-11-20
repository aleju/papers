# Paper

* **Title**: Progressive Growing of GANs for Improved Quality, Stability, and Variation
* **Authors**: Tero Karras, Timo Aila, Samuli Laine, Jaakko Lehtinen
* **Link**: http://research.nvidia.com/sites/default/files/pubs/2017-10_Progressive-Growing-of/karras2017gan-paper.pdf
* **Tags**: Neural Network, GAN
* **Year**: 2017

# See also

* [Video](https://www.youtube.com/watch?v=XOxxPcy5Gr4)
  * You have to pause the video on the CelebA images to see the generated artifacts/errors. The motion makes it much harder to see these.

# Tiny Summary

They suggest a new method to train GANs.
They start training them at low resolution (4x4), wait until "convergence", then add more convolutions to the existing model to generate and discriminate higher resolutions.
Each new block of convolutions is slowly blended in, instead of being added from one batch to the next.
Combined with two new normalization techniques, they get good-looking images at up to `1024x1024` on their new CelebA-HQ dataset (CelebA in high resolution).
They also suggest a new scoring method based on the approximated Wasserstein distance between real and generated image patches.
According to that score, their progressive training method improves results significantly.

# Summary

* What
  * They suggest a new, progressive training method for GANs.
    The method enables the training of high resolution GANs (1024x1024) that still produce good-looking, diverse images.
    * They also introduce two new normalization techniques.
  * They also suggest a new method to estimate/score the quality of the generated images.
  * They introduce CelebA-HQ, a variation of CelebA containing high resolution images.

* How
  * Progressive growing/training
    * They train their GANs resolution by resolution, starting with 4x4 and going up to 1024x1024 (a bit similar to LAPGAN).
    * Visualization:
      * ![progressive growing](images/Progressive_Growing_of_GANs/progressive_growing.jpg?raw=true "progressive growing")
    * Initially, their generator produces 4x4 images and the discriminator receives 4x4 images.
    * Once training at 4x4 does not improve any more (measured by their new score, see below), they add an upscaling module (to 8x8) to the generator and add a downscaling one to the discriminator.
    * They don't switch to the added convolutions instantly/suddenly,
      but give the model a grace period during which the upscaled features are computed from `(1-alpha)*A + alpha*B`,
      where `A` are the features after just upscaling, `B` are the features after upscaling AND the convolutions
      and `alpha` is the overlay factor, which is gradually increased over time.
      This is done for both the generator and the discriminator and at all resolutions.
    * Visualization:
      * ![add new block](images/Progressive_Growing_of_GANs/add_new_block.jpg?raw=true "add new block")
    * Note that all layers are always trained (after they were added to the models). Training for the earlier layers does *not* stop.
    * Training in this way focuses most of the computation on the earlier resolutions.
      It also seems to increase stability, as the model does not have to learn all features of all resolutions at the same time.
  * Minibatch Standard Deviation
    * They try to improve diversity by adding a method very similar to minibatch discrimination.
    * They compute the standard deviation of each feature per spatial location (for one of the disciminator's last layers).
    * They do this per example in each minibatch, resulting in `B*H*W*C` standard deviations. (B = batch size, H = height, W = width, C = channels/filters)
    * They average these values to one value, then replicate them to size `H*W` and concatenate that to the layer's output.
    * This adds a channel with one constant value to each example in the minibatch. The value is the same for all examples.
  * Equalized Learning Rate
    * They use Adam for their training.
    * Adam updates weights roughly based on `mean(gradient)/variance(gradient)` (per weight).
    * They argue that this has the downside of equalizing all weight's stepsizes.
      But some weights might require larger stepsizes and other smaller ones (large/small "dynamic range").
      As a result, the learning rate will be too small for some weights and too large for others.
    * To evade this problem, they first stop using modern weight initialization techniques and instead simply sample weights from the standard normal distribution `N(0,1)`.
    * Then, they rescale each weight `w_i` *continuously during runtime* to `w_i/c`, where `c` is the per-layer normalization from He's initializer. (TODO exact formula for c?)
      * (This looks an aweful lot like [weight normalization](On_The_Effects_of_BN_and_WN_in_GANs.md).)
    * Using simpler weight initialization equalizes the dynamic range of parameters. Doing the normalization then fixes problems related to the simpler weight initialization.
  * Pixelwise Feature Vector Normalization in the Generator
    * They argue that collapses in GANs come from the discriminator making some temporary error, leading to high gradients, leading to bad outputs of the generator, leading to more problems in the discriminator and ultimately making both spiral out of control.
    * They fix this by normalizing feature vectors in the generator, similar to local response normalization.
    * They apply the following equation in the generator (per spatial location `(x, y)` with `N` = number of filters):
      * ![pixelwise normalization](images/Progressive_Growing_of_GANs/pixelwise.jpg?raw=true "pixelwise normalization")
  * Scoring Images
    * They suggest a new method to score images generated by the generator.
    * They perform the following steps:
      1. Sample 16384 images from the generator and the dataset.
      2. Build a Laplacian Pyramid of each image.
         It begins at a 16x16 resolution of the image and progressively doubles that until the final image resolution.
         Each level of the pyramid only contains the difference between the sum of the previous scales and the final image (i.e. each step is a difference image, containing a frequency band).
      3. Sample per image 128 7x7 neighbourhoods/patches (randomly?) from each pyramid level.
      4. Per image set (generator/real) and pyramid level, compute the mean and standard deviations of each color channels of the sampled patches.
      5. Normalize each patch with respect to the computed means and standard deviations.
      6. Use Sliced Wasserstein Distance (SWD) to compute the similarity between the image sets (generator/real).
    * The result is one value. Lower values are better.
  * CelebA-HQ
    * They derive from CelebA images a new dataset containing 30k 1024x1024 images of celebrity faces.
    * They use a convolutional autoencoder to remove JPEG artifacts from the CelebA images.
    * They use an adversarially-trained superresolution model to upscale the images.
    * They crop faces from the dataset based on their facial landmarks, so that each final face has a normalized position and rotation.
    * They rescale the images to 1024x1024 using bilinear sampling and box filters.
    * They manually select the 30k best looking images.
  * Other stuff
    * They use Adam for training (alpha=0.001, beta1=0, beta2=0.99).
    * They use the WGAN-WP method for training, but LSGAN also works.
      * They set `gamma` to 750 (from 1) for CIFAR-10, incentivizing fast transitions.
      * They also add regularization loss on the discriminator, punishing outputs that are very far away from 0.
    * Their model for CelebA-HQ training is similar to a standard DCGAN model.
      The generator uses two convolutions after each upscaling, the discriminator analogously two convolutions after each downscaling.
      They start with 512 filters in the generator and end in 16 (before the output) - same for the discriminator.
      They use leaky ReLUs in the generator and discriminator.
      They remove batch normalization everywhere.

* Results
  * Scores
    * Results, according to their new scoring measure (Sliced Wasserstein Distance) and MS-SSIM measure:
      * ![scores](images/Progressive_Growing_of_GANs/scores.jpg?raw=true "scores")
    * So *progressive growing* (b) significantly improves results.
      Same -- to a smaller degree -- for *minibatch standard deviation* (e), *equalized learning rate* (f) and *pixelwise normalization* (g).
      *Minibatch discrimination* worsened the results.
      Using small batch sizes also worsened the results.
      In (d) they "adjusted the hyperparameters" (??) and removed batch normalization.
  * They generate 1024x1024 CelebA images, while maintaining pixelwise quality compared to previous models.
  * They achieve an Inception Score of 8.80 on CIFAR-10. Images look improved.
  * CelebA-HQ example results:
    * ![celeba-hq](images/Progressive_Growing_of_GANs/celeba.jpg?raw=true "celeba-hq")
  * LSUN dining room, horse, kitchen, churches:
    * ![dining room](images/Progressive_Growing_of_GANs/dining_room.jpg?raw=true "dining room")
    * ![horse kitchen](images/Progressive_Growing_of_GANs/horse_kitchen.jpg?raw=true "horse kitchen")
    * ![churches](images/Progressive_Growing_of_GANs/churches.jpg?raw=true "churches")
    * ![bedrooms](images/Progressive_Growing_of_GANs/bedrooms.jpg?raw=true "bedrooms")

