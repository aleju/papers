# Paper

* **Title**: Adversarially Learned Inference
* **Authors**: Vincent Dumoulin, Ishmael Belghazi, Ben Poole, Alex Lamb, Martin Arjovsky, Olivier Mastropietro, Aaron Courville
* **Link**: http://arxiv.org/abs/1606.00704
* **Tags**: Neural Network, GAN, variational
* **Year**: 2016

# Summary

* What
  * They suggest a new architecture for GANs.
  * Their architecture adds another Generator for a reverse branch (from images to noise vector `z`).
  * Their architecture takes some ideas from VAEs/variational neural nets.
  * Overall they can improve on the previous state of the art (DCGAN).

* How
  * Architecture
    * Usually, in GANs one feeds a noise vector `z` into a Generator (G), which then generates an image (`x`) from that noise.
    * They add a reverse branch (G2), in which another Generator takes a real image (`x`) and generates a noise vector `z` from that.
      * The noise vector can now be viewed as a latent space vector.
    * Instead of letting G2 generate *discrete* values for `z` (as it is usually done), they instead take the approach commonly used VAEs and use *continuous* variables instead.
      * That is, if `z` represents `N` latent variables, they let G2 generate `N` means and `N` variances of gaussian distributions, with each distribution representing one value of `z`.
      * So the model could e.g. represent something along the lines of "this face looks a lot like a female, but with very low probability could also be male".
  * Training
    * The Discriminator (D) is now trained on pairs of either `(real image, generated latent space vector)` or `(generated image, randomly sampled latent space vector)` and has to tell them apart from each other.
    * Both Generators are trained to maximally confuse D.
      * G1 (from `z` to `x`) confuses D maximally, if it generates new images that (a) look real and (b) fit well to the latent variables in `z` (e.g. if `z` says "image contains a cat", then the image should contain a cat).
      * G2 (from `x` to `z`) confuses D maximally, if it generates good latent variables `z` that fit to the image `x`.
    * Continuous variables
      * The variables in `z` follow gaussian distributions, which makes the training more complicated, as you can't trivially backpropagate through gaussians.
      * When training G1 (from `z` to `x`) the situation is easy: You draw a random `z`-vector following a gaussian distribution (`N(0, I)`). (This is basically the same as in "normal" GANs. They just often use uniform distributions instead.)
      * When training G2 (from `x` to `z`) the situation is a bit harder.
        * Here we need to use the reparameterization trick here.
        * That roughly means, that G2 predicts the means and variances of the gaussian variables in `z` and then we draw a sample of `z` according to exactly these means and variances.
        * That sample gives us discrete values for our backpropagation.
        * If we do that sampling often enough, we get a good approximation of the true gradient (of the continuous variables). (Monte Carlo approximation.)

* Results
  * Images generated based on Celeb-A dataset:
    * ![Celeb-A samples](images/Adversarially_Learned_Inference__celeba-samples.png?raw=true "Celeb-A samples")
  * Left column per pair: Real image, right column per pair: reconstruction (`x -> z` via G2, then `z -> x` via G1)
    * ![Celeb-A reconstructions](images/Adversarially_Learned_Inference__celeba-reconstructions.png?raw=true "Celeb-A reconstructions")
  * Reconstructions of SVHN, notice how the digits often stay the same, while the font changes:
    * ![SVHN reconstructions](images/Adversarially_Learned_Inference__svhn-reconstructions.png?raw=true "SVHN reconstructions")
  * CIFAR-10 samples, still lots of errors, but some quite correct:
    * ![CIFAR10 samples](images/Adversarially_Learned_Inference__cifar10-samples.png?raw=true "CIFAR10 samples")

