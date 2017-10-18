# Paper

* **Title**: Unsupervised Image-to-Image Translation Networks
* **Authors**: Ming-Yu Liu, Thomas Breuel, Jan Kautz
* **Link**: https://arxiv.org/abs/1703.00848
* **Tags**: Neural Network, GAN
* **Year**: 2017

# Summary

* What
  * They present a method to learn mapping functions that transform images from one style to another style. (E.g. photos from daylight to nighttime.)
  * Their method only requires example images for both styles (i.e. class labels per image).

* How
  * Architecture
    * Their method is based on VAEs (i.e. autoencoders) and GANs.
    * Their architecture is kinda similar to an autoencoder.
    * For an image style `A`, an encoder `E` first transform an image to a vector representation `z`.
      Then a generator `G` transforms `z` into an image.
    * There are two encoders (`E1`, `E2`), one per image style (`A`, `B`).
    * There are two generators (`G1`, `G2`), one per image style (`A`, `B`).
    * There are two discriminators (`D1`, `D2`), one per generator (and therefore style).
    * An image can be changed in style from `A` to `B` using e.g. `G2(E1(I_A))`.
    * The weights of the encoders are mostly tied/shared. Only the last layers are not-shared.
    * The weights of the generators are mostly tied/shared. Only the last layers are not-shared.
    * They use 3 convs + 4 residual blocks for the encoders and 4 residual blocks + 3 transposes convs for the generators.
      They use normal convs for the discriminators. Nonlinearities are LeakyReLUs.
    * The encoders are VAEs and trained with common VAE-losses (i.e. lower bound optimization).
      However, they only predict mean values per component in `z`, not variances.
      The variances are all `1`.
    * Visualization of the architecture:
      * ![architecture](images/Unsupervised_Image-to-Image_Translation_Networks/architecture.jpg?raw=true "architecture")
  * Loss
    * Their loss consists of three components:
      * VAE-loss: Reconstruction loss (absolute distance) and KL term on z (to keep it close to the standard normal distribution).
        Most weight is put on the reconstruction loss.
      * GAN-loss: Standard as in other GANs, i.e. cross-entropy.
      * Cycle-Consistency-loss: For an image `I_A`, it is expected to look the same after switching back and forth between image styles, i.e.
        `I_A = G1(E2( G2(E1(I_A)) ))` (switch from style A to B, then from B to A).
        The cycle consistency loss uses a reconstruction loss and two KL-terms (one for the first E(.) and one for the second).

* Results
  * When testing on the (satellite) map dataset:
    * Weight sharing between encoders and between generators improved accuracy.
    * The cycle consistency loss improved accuracy.
    * Using 4-6 layers (as opposed to just 3) in the discriminator improved accuracy.
  * Translations that added details (e.g. night to day) were harder for the model.
  * After training, the features from each discriminator seem to be quite good for the respective dataset (i.e. unsupervised learned features).
  * Example translations:
    * ![examples](images/Unsupervised_Image-to-Image_Translation_Networks/examples.jpg?raw=true "examples")
