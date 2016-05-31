# Paper

* **Title**: Rank Ordered Autoencoders
* **Authors**: Paul Bertens
* **Link**: https://arxiv.org/abs/1605.01749
* **Tags**: Neural Network, autoencoder, rank order
* **Year**: 2016

# Summary

* What
  * Autoencoders typically have some additional criterion that pushes them towards learning meaningful representations.
    * E.g. L1-Penalty on the code layer (z), Dropout on z, Noise on z.
    * Often, representations with sparse activations are considered meaningful (so that each activation reflects are clear concept).
  * This paper introduces another technique that leads to sparsity.
  * They use a rank ordering on z.
  * The first (according to the ranking) activations have to do most of the reconstruction work of the data (i.e. image).

* How

* Results

![Examples](images/Accurate_Image_Super-Resolution__examples.png?raw=true "Examples")

*Super-resolution quality of their model (top, bottom is a competing model).*
