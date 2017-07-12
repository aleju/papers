# Paper

* **Title**: BEGAN: Boundary Equilibrium Generative Adversarial Networks
* **Authors**: David Berthelot, Thomas Schumm, Luke Metz
* **Link**: https://arxiv.org/abs/1703.10717
* **Tags**: Neural Network, GAN
* **Year**: 2017

# See also

* [Detailed Summary](https://blog.heuritech.com/2017/04/11/began-state-of-the-art-generation-of-faces-with-generative-adversarial-networks/)
* [Tensorflow implementation](https://github.com/carpedm20/BEGAN-tensorflow)

# Summary

* What
  * They suggest a GAN algorithm that is based on an autoencoder with Wasserstein distance.
  * Their method generates highly realistic human faces.
  * Their method has a convergence measure, which reflects the quality of the generates images.
  * Their method has a diversity hyperparameter, which can be used to set the tradeoff between image diversity and image quality.

* How
  * Like other GANs, their method uses a generator G and a discriminator D.
  * Generator
    * The generator is fairly standard.
    * It gets a noise vector `z` as input and uses upsampling+convolutions to generate images.
    * It uses ELUs and no BN.
  * Discriminator
    * The discriminator is a full autoencoder (i.e. it converts input images to `8x8x3` tensors, then reconstructs them back to images).
    * It has skip-connections from the `8x8x3` layer to each upsampling layer.
    * It also uses ELUs and no BN.
  * Their method now has the following steps:
    1. Collect real images `x_real`.
    2. Generate fake images `x_fake = G(z)`.
    3. Reconstruct the real images `r_real = D(x_real)`.
    4. Reconstruct the fake images `r_fake = D(x_fake)`.
    5. Using an Lp-Norm (e.g. L1-Norm), compute the reconstruction loss of real images `d_real = Lp(x_real, r_real)`.
    6. Using an Lp-Norm (e.g. L1-Norm), compute the reconstruction loss of fake images `d_fake = Lp(x_fake, r_fake)`.
    7. The loss of D is now `L_D = d_real - d_fake`.
    8. The loss of G is now `L_G = -L_D`.
  * About the loss
    * `r_real` and `r_fake` are really losses (e.g. L1-loss or L2-loss). In the paper they use `L(...)` for that. Here they are referenced as `d_*` in order to avoid confusion.
    * The loss `L_D` is based on the Wasserstein distance, as in WGAN.
    * `L_D` assumes, that the losses `d_real` and `d_fake` are normally distributed and tries to move their mean values. Ideally, the discriminator produces very different means for real/fake images, while the generator leads to very similar means.
    * Their formulation of the Wasserstein distance does not require K-Lipschitz functions, which is why they don't have the weight clipping from WGAN.
  * Equilibrium
    * The generator and discriminator are at equilibrium, if `E[r_fake] = E[r_real]`. (That's undesirable, because it means that D can't differentiate between fake and real images, i.e. G doesn't get a proper gradient any more.)
    * Let `g = E[r_fake] / E[r_real]`, then:
      * Low `g` means that `E[r_fake]` is low and/or `E[r_real]` is high, which means that real images are not as well reconstructed as fake images. This means, that the discriminator will be more heavily trained towards reconstructing real images correctly (as that is the main source of error).
      * High `g` conversely means that real images are well reconstructed (compared to fake ones) and that the discriminator will be trained more towards fake ones.
      * `g` gives information about how much G and D should be trained each (so that none of the two overwhelms the other).
    * They introduce a hyperparameter `gamma` (from interval `[0,1]`), which reflects the target value of the balance `g`.
    * Using `gamma`, they change their losses `L_D` and `L_G` slightly:
      * `L_D = d_real - k_t d_fake`
      * `L_G = r_fake`
      * `k_t+1 = k_t + lambda_k (gamma d_real - d_fake)`.
    * `k_t` is a control term that controls how much D is supposed to focus on the fake images. It changes with every batch.
    * `k_t` is clipped to `[0,1]` and initialized at `0` (max focus on reconstructing real images).
    * `lambda_k` is like the learning rate of the control term, set to `0.001`.
    * Note that `gamma d_real - d_fake = 0 <=> gamma d_real = d_fake <=> gamma = d_fake / d_real`.
  * Convergence measure
    * They measure the convergence of their model using `M`:
    * `M = d_real + |gamma d_real - d_fake|`
    * `M` goes down, if `d_real` goes down (D becomes better at autoencoding real images).
    * `M` goes down, if the difference in reconstruction error between real and fake images goes down, i.e. if G becomes better at generating fake images.
  * Other
    * They use Adam with learning rate 0.0001. They decrease it by a factor of 2 whenever M stalls.
    * Higher initial learning rate could lead to model collapse or visual artifacs.
    * They generate images of max size 128x128.
    * They don't use more than 128 filters per conv layer.

* Results
  * NOTES:
    * Below example images are NOT from generators trained on CelebA. They used a custom dataset of celebrity images. They don't show any example images from the dataset. The generated images look like there is less background around the faces, making the task easier.
    * Few example images. Unclear how much cherry picking was involved. Though the results from the tensorflow example (see like at top) make it look like the examples are representative (aside from speckle-artifacts).
    * No LSUN Bedrooms examples. Human faces are comparatively easy to generate.
  * Example images at 128x128:
    * ![Examples](images/BEGAN__examples.jpg?raw=true "Examples")
  * Effect of changing the target balance `gamma`:
    * ![Examples gamma](images/BEGAN__examples_gamma.jpg?raw=true "Examples gamma")
    * High gamma leads to more diversity at lower quality.
  * Interpolations:
    * ![Interpolations](images/BEGAN__interpolations.jpg?raw=true "Interpolations")
  * Convergence measure `M` and associated image quality during the training:
    * ![M](images/BEGAN__convergence.jpg?raw=true "M")
