# Paper

* **Title**: DRAW: A Recurrent Neural Network For Image Generation
* **Authors**: Karol Gregor, Ivo Danihelka, Alex Graves, Danilo Jimenez Rezende, Daan Wierstra
* **Link**: http://arxiv.org/abs/1502.04623
* **Tags**: Neural Network, generative models, recurrent, attention
* **Year**: 2015

# Summary



----------

# Rough chapter-wise notes

* 1. Introduction
  * The natural way to draw an image is in a step by step way (add some lines, then add some more, etc.).
  * Most generative neural networks however create the image in one step.
  * That removes the possibility of iterative self-correction, is hard to scale to large images and makes the image generation process dependent on a single latent distribution (input parameters).
  * The DRAW architecture generates images in multiple steps, allowing refinements/corrections.
  * DRAW is based on varational autoencoders: An encoder compresses images to codes and a decoder generates images from codes.
  * The loss function is a variational upper bound on the log-likelihood of the data.
  * DRAW uses recurrance to generate images step by step.
  * The recurrance is combined with attention via partial glimpses/foveations (i.e. the model sees only a small part of the image).
  * Attention is implemented in a differentiable way in DRAW.

* 2. The DRAW Network
  * The DRAW architecture is based on variational autoencoders:
    * Encoder: Compresses an image to latent codes, which represent the information contained in the image.
    * Decoder: Transforms the codes from the encoder to images (i.e. defines a distribution over images which is conditioned on the distribution of codes).
  * Differences to variational autoencoders:
    * Encoder and decoder are both recurrent neural networks.
    * The encoder receives the previous output of the decoder.
    * The decoder writes several times to the image array (instead of only once).
    * The encoder has an attention mechanism. It can make a decision about the read location in the input image.
    * The decoder has an attention mechanism. It can make a decision about the write location in the output image.
  * 2.1 Network architecture
    * They use LSTMs for the encoder and decoder.
    * The encoder generates a vector.
    * The decoder generates a vector.
    * The encoder receives at each time step the image and the output of the previous decoding step.
    * The hidden layer in between encoder and decoder is a distribution Q(Zt|ht^enc), which is a diagonal gaussian.
    * The mean and standard deviation of that gaussian is derived from the encoder's output vector with a linear transformation.
    * Using a gaussian instead of a bernoulli distribution enables the use of the reparameterization trick. That trick makes it straightforward to backpropagate "low variance stochastic gradients of the loss function through the latent distribution".
    * The decoder writes to an image canvas. At every timestep the vector generated the the decoder is added to that canvas.
    * 
