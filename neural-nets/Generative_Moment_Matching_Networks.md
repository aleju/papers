# Paper

* **Title**: Generative Moment Matching Networks
* **Authors**: Yujia Li, Kevin Swersky, Richard Zemel
* **Link**: http://arxiv.org/abs/1502.02761
* **Tags**: Neural Network, generative model
* **Year**: 2015

# Summary

* What
  * Generative Moment Matching Networks (GMMN) are generative models that use maximum mean discrepancy (MMD) for their objective function.
  * MMD is a measure of how similar two datasets are (here: generated dataset and training set).
  * GMMNs are similar to GANs, but they replace the Discriminator with the MMD measure, making their optimization more stable.

* How
  * MMD calculates a similarity measure by comparing statistics of two datasets with each other.
  * MMD is calculated based on samples from the training set and the generated dataset.
  * A kernel function is applied to pairs of these samples (thus the statistics are acutally calculated in high-dimensional spaces). The authors use Gaussian kernels.
  * MMD can be approximated using a small number of samples.
  * MMD is differentiable and therefor can be used as a standard loss function.
  * They train two models:
    * GMMN: Noise vector input (as in GANs), several ReLU layers into one sigmoid layer. MMD as the loss function.
    * GMMN+AE: Same as GMMN, but the sigmoid output is not an image, but instead the code that gets fed into an autoencoder's (AE) decoder. The AE is trained separately on the dataset. MMD is backpropagated through the decoder and then the GMMN. I.e. the GMMN learns to produce codes that let the decoder generate good looking images.

![Formula](images/Generative_Moment_Matching_Networks__formula.png?raw=true "Formula")

*MMD formula, where `x_i` is a training set example and `y_i` a generated example.*


![Architectures](images/Generative_Moment_Matching_Networks__architectures.png?raw=true "Architectures")

*Architectures of GMMN (left) and GMMN+AE (right).*


* Results
  * They tested only on MNIST and TFD (i.e. datasets that are well suited for AEs...).
  * Their GMMN achieves similar log likelihoods compared to other models.
  * Their GMMN+AE achieves better log likelihoods than other models.
  * GMMN+AE produces good looking images.
  * GMMN+AE produces smooth interpolations between images.

![Interpolations](images/Generative_Moment_Matching_Networks__interpolations.png?raw=true "Interpolations")

*Generated TFD images and interpolations between them.*


--------------------

# Rough chapter-wise notes

* (1) Introduction
  * Sampling in GMMNs is fast.
  * GMMNs are similar to GANs.
  * While the training objective in GANs is a minimax problem, in GMMNs it is a simple loss function.
  * GMMNs are based on maximum mean discrepancy. They use that (implemented via the kernel trick) as the loss function.
  * GMMNs try to generate data so that the moments in the generated data are as similar as possible to the moments in the training data.
  * They combine GMMNs with autoencoders. That is, they first train an autoencoder to generate images. Then they train a GMMN to produce sound code inputs to the decoder of the autoencoder.

* (2) Maximum Mean Discrepancy
  * Maximum mean discrepancy (MMD) is a frequentist estimator to tell whether two datasets X and Y come from the same probability distribution.
  * MMD estimates basic statistics values (i.e. mean and higher order statistics) of both datasets and compares them with each other.
  * MMD can be formulated so that examples from the datasets are only used for scalar products. Then the kernel trick can be applied.
  * It can be shown that minimizing MMD with gaussian kernels is equivalent to matching all moments between the probability distributions of the datasets.

* (4) Generative Moment Matching Networks
  * Data Space Networks
    * Just like GANs, GMMNs start with a noise vector that has N values sampled uniformly from [-1, 1].
    * The noise vector is then fed forward through several fully connected ReLU layers.
    * The MMD is differentiable and therefor can be used for backpropagation.
  * Auto-Encoder Code Sparse Networks
    * AEs can be used to reconstruct high-dimensional data, which is a simpler task than to learn to generate new data from scratch.
    * Advantages of using the AE code space:
      * Dimensionality can be explicitly chosen.
      * Disentangling factors of variation.
    * They suggest a combination of GMMN and AE. They first train an AE, then they train a GMMN to generate good codes for the AE's decoder (based on MMD loss).
    * For some reason they use greedy layer-wise pretraining with later fine-tuning for the AE, but don't explain why. (That training method is outdated?)
    * They add dropout to their AE's encoder to get a smoother code manifold.
  * Practical Considerations
    * MMD has a bandwidth parameter (as its based on RBFs). Instead of chosing a single fixed bandwidth, they instead use multiple kernels with different bandwidths (1, 5, 10, ...), apply them all and then sum the results.
    * Instead of `MMD^2` loss they use `sqrt(MMD^2)`, which does not go as fast to zero as raw MMD, thereby creating stronger gradients.
    * Per minibatch they generate a small number of samples und they pick a small number of samples from the training set. They then compute MMD for these samples. I.e. they don't run MMD over the whole training set as that would be computationally prohibitive.

* (5) Experiments
  * They trained on MNIST and TFD.
  * They used an GMMN with 4 ReLU layers and autoencoders with either 2/2 (encoder, decoder) hidden sigmoid layers (MNIST) or 3/3 (TFD).
  * They used dropout on the encoder layers.
  * They used layer-wise pretraining and finetuning for the AEs.
  * They tuned most of the hyperparameters using bayesian optimization.
  * They use minibatch sizes of 1000 and compute MMD based on those (i.e. based on 2000 points total).
  * Their GMMN+AE model achieves better log likelihood values than all competitors. The raw GMMN model performs roughly on par with the competitors.
  * Nearest neighbor evaluation indicates that it did not just memorize the training set.
  * The model learns smooth interpolations between digits (MNIST) and faces (TFD).
