# Paper

* **Title**: Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks
* **Authors**: Alec Radford, Luke Metz, Soumith Chintala
* **Link**: http://arxiv.org/abs/1511.06434
* **Tags**: Neural Network, GAN, generative model, laplacian pyramid
* **Year**: 2016

# Summary


# Rough chapter-wise notes

* Introduction
  * For unsupervised learning, they propose to use to train a GAN and then reuse the weights of D.
  * GANs have traditionally been hard to train.

* Approach and model architecture
  * They use for D an convnet without linear layers, withput pooling layers (only strides), LeakyReLUs and Batch Normalization.
  * They use for G ReLUs (hidden layers) and Tanh (output).
