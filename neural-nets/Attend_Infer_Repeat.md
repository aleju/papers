# Paper

* **Title**: Attend, Infer, Repeat: Fast Scene Understanding with Generative Models
* **Authors**: S. M. Ali Eslami, Nicolas Heess, Theophane Weber, Yuval Tassa, Koray Kavukcuoglu, Geoffrey E. Hinton
* **Link**: http://arxiv.org/abs/1603.08575
* **Tags**: Neural Network, VAE, attention, recurrent, LSTM, unsupervised
* **Year**: 2016

# Summary

* What
  * AIR (attend, infer, repeat) is a recurrent autoencoder architecture to transform images into latent representations object by object.
  * As an autoencoder it is unsupervised.
  * The latent representation is generated in multiple time steps.
  * Each time step is intended to encode information about exactly one object in the image.
  * The information encoded for each object is (mostly) a what-where information, i.e. which class the object has and where (in 2D: translation, scaling) it is shown.
  * AIR has a dynamic number of time step. After encoding one object the model can decide whether it has encoded all objects or whether there is another one to encode. As a result the latent layer size is not fixed.
  * AIR uses an attention mechanism during the encoding to focus on each object.

* How
  * At its core, AIR is a variational autoencoder.
  * It maximizes lower bounds on the error instead of using a "classic" reconstruction error (like MSE on the euclidean distance).
  * It has an encoder and a decoder.
  * The model uses a recurrent architecture via an LSTM.
  * It (ideally) encodes/decodes one object per time step.
  * Encoder
    * The encoder receives the image and generates latent information for one object (what object, where it is).
    * At the second timestep it receives the image, the previous timestep's latent information and the previous timestep's hidden layer. It then generates another latent information (for another object).
    * And so on.
  * Decoder
    * The decoder receives latent information from the encoder (timestep by timestep) and treats it as a what-where information when reconstructing the images.
      * It takes the what-part and uses a "normal" decoder to generate an image that shows the object.
      * It takes the where-part and the generated image and feeds both into a spatial transformer, which then transforms the generated image by translating or rotating it.
  * Dynamic size
    * AIR makes use of a dynamically sized latent layer. It is not necessarily limited to a fixed number of time steps.
    * Implementation: Instead of just letting the encoder generate what-where information, the encoder also generates a "present" information, which is 0 or 1. If it is 1, the reccurence will continue with encoding and decoding another object. Otherwise it will stop.
  * Attention
    * To add an attention mechanism, AIR first uses the LSTM's hidden layer to generate "where" and "present" information per object.
    * It stops if the "present" information is 0.
    * Otherwise it uses the "where" information to focus on the object using a spatial transformer. The object is then encoded to the "what" information.

* Results
  * On a dataset of images, each containing multiple MNIST digits, AIR learns to accurately count the digits and estimate their position and scale.
  * When AIR is trained on images of 0 to 2 digits and tested on images containing 3 digits it performs poorly.
  * When AIR is trained on images of 0, 1 or 3 digits and tested on images containing 2 digits it performs mediocre.
  * DAIR performs well on both tasks. Likely because it learns to remove each digit from the image after it has investigated it.
  * When AIR is trained on 0 to 2 digits and a second network is trained (separately) to work with the generated latent layer (trained to sum the shown digits and rate whether they are shown in ascending order), then that second network reaches high accuracy with relatively few examples. That indicates usefulness for unsupervised learning.
  * When AIR is trained on a dataset of handwritten characters from different alphabets, it learns to represent distinct strokes in its latent layer.
  * When AIR is trained in combination with a renderer (inverse graphics), it is able to accurately recover latent parameters of rendered objects - better than supervised networks. That indicates usefulness for robots which have to interact with objects.


![Architecture](images/Attend_Infer_Repeat__architecture.png?raw=true "Architecture.")

*AIR architecture for MNIST. Left: Decoder for two objects that are each first generated (y_att) and then fed into a Spatial Transformer (y) before being combined into an image (x). Middle: , Right: Encoder with multiple time steps that generates what-where information per object and stops when the "present" information (z_pres) is 0. Right: Combination of both for MNIST with Spatial Transformer for the attention mechanism (top left).*


![DAIR Architecture](images/Attend_Infer_Repeat__architecture_dair.png?raw=true "DAIR Architecture.")

*Encoder with DAIR architecture. DAIR modifies the image after every timestep (e.g. to remove objects that were already encoded).*


--------------------

# Rough chapter-wise notes

* (1) Introduction
  * Assumption: Images are made up of distinct objects. These objects have visual and physical properties.
  * They developed a framework for efficient inference in images (i.e. get from the image to a latent representation of the objects, i.e. inverse graphics).
  * Parts of the framework: High dimensional representations (e.g. object images), interpretable latent variables (e.g. for rotation) and generative processes (to combine object images with latent variables).
  * Contributions:
    * A scheme for efficient variational inference in latent spaces of variable dimensionality.
      * Idea: Treat inference as an iterative process, implemented via an RNN that looks at one object at a time and learns an appropriate number of inference steps. (Attend-Infer-Repeat, AIR)
      * End-to-end training via amortized variational inference (continuous variables: gradient descent, discrete variables: black-box optimization).
    * AIR allows to train generative models that automatically learn to decompose scenes.
    * AIR allows to recover objects and their attributes from rendered 3D scenes (inverse rendering).

* (2) Approach
  * Just like in VAEs, the scene interpretation is treated with a bayesian approach.
  * There are latent variables `z` and images `x`.
  * Images are generated via a probability distribution `p(x|z)`.
  * This can be reversed via bayes rule to `p(x|z) = p(x)p(z|x) / p(z)`, which means that `p(x|z)p(z) / p(x) = p(z|x)`.
  * The prior `p(z)` must be chosen and captures assumptions about the distributions of the latent variables.
  * `p(x|z)` is the likelihood and represents the model that generates images from latent variables.
  * They assume that there can be multiple objects in an image.
  * Every object gets its own latent variables.
  * A probability distribution p(x|z) then converts each object (on its own) from the latent variables to an image.
  * The number of objects follows a probability distribution `p(n)`.
  * For the prior and likelihood they assume two scenarios:
    * 2D: Three dimensions for X, Y and scale. Additionally n dimensions for its shape.
    * 3D: Dimensions for X, Y, Z, rotation, object identity/category (multinomial variable). (No scale?)
  * Both 2D and 3D can be separated into latent variables for "where" and "what".
  * It is assumed that the prior latent variables are independent of each other.
  * (2.1) Inference
    * Inference for their model is intractable, therefore they use an approximation `q(z,n|x)`, which minizes `KL(q(z,n|x)||p(z,n|x))`, i.e. KL(approximation||real) using amortized variational approximation.
    * Challenges for them:
      * The dimensionality of their latent variable layer is a random variable p(n) (i.e. no static size.).
      * Strong symmetries.
    * They implement inference via an RNN which encodes the image object by object.
    * The encoded latent variables can be gaussians.
    * They encode the latent layer length `n` via a vector (instead of an integer). The vector has the form of `n` ones followed by one zero.
    * If the length vector is `#z` then they want to approximate `q(z,#z|x)`.
    * That can apparently be decomposed into `<product> q(latent variable value i, #z is still 1 at i|x, previous latent variable values) * q(has length n|z,x)`.
    * So instead of computing `#z` once, they instead compute at every time step whether there is another object in the image, which indirectly creates a chain of ones followed by a zero (the `#z` vector).
  * (2.2) Learning
    * The parameters theta (`p`, latent variable -> image) and phi (`q`, image -> latent variables) are jointly optimized.
    * Optimization happens by maximizing a lower bound `E[log(p(x,z,n) / q(z,n|x))]` called the negative free energy.
    * (2.2.1) Parameters of the model theta
      * Parameters theta of log(p(x,z,n)) can easily be obtained using differentiation, so long as z and n are well approximated.
      * The differentiation of the lower bound with repsect to theta can be approximated using Monte Carlo methods.
    * (2.2.2) Parameters of the inference network phi
      * phi are the parameters of q, i.e. of the RNN that generates z and #z in i timesteps.
      * At each timestep (i.e. per object) the RNN generates three kinds of information: What (object), where (it is), whether it is present (i <= n).
      * Each of these information is represented via variables. These variables can be discrete or continuous.
      * When differentiating w.r.t. a continuous variable they use the reparameterization trick.
      * When differentiating w.r.t. a discrete variable they use the likelihood ratio estimator.

* (3) Models and Experiments
  * The RNN is implemented via an LSTM.
  * DAIR
    * The "normal" AIR model uses at every time step the image and the RNN's hidden layer to generate the next latent information (what object, where it is and whether it is present).
    * DAIR uses that latent information to change the image at every time step and then use the difference (D) image for the next time step, i.e. DAIR can remove an object from the image after it has generated latent variables for it.
  * (3.1) Multi-MNIST
    * They generate a dataset of images containing multiple MNIST digits.
    * Each image contains 0 to 2 digits.
    * AIR is trained on the dataset.
    * It learns without supervision a good attention scanning policy for the images (to "hit" all digits), to count the digits visible in the image and to use a matching number of time steps.
    * During training, the model seems to first learn proper reconstruction of the digits and only then to do it with as few timesteps as possible.
    * (3.1.1) Strong Generalization
      * They test the generalization capabilities of AIR.
      * *Extrapolation task*: They generate images with 0 to 2 digits for training, then test on images with 3 digits. The model is unable to correctly count the digits (~0% accuracy).
      * *Interpolation task*: They generate images with 0, 1 or 3 digits for training, then test on images with 2 digits. The model performs OK-ish (~60% accuracy).
      * DAIR performs in both cases well (~80% for extrapolation, ~95% accuracy for interpolation).
    * (3.1.2) Representational Power
      * They train AIR on images containing 0, 1 or 2 digits.
      * Then they train a second network. That network takes the output of the first one and computes a) the sum of the digits and b) estimates whether they are shown in ascending order.
      * Accuracy for both tasks is ~95%.
      * The network reaches that accuracy significantly faster than a separately trained CNN (i.e. requires less labels / is more unsupervised).
  * (3.2) Omniglot
    * They train AIR on the Omniglot dataset (1.6k handwritten characters from 50 alphabets).
    * They allow the model to use up to 4 timesteps.
    * The model learns to reconstruct the images in timesteps that resemble strokes.
  * (3.3) 3D Scenes
    * Here, the generator p(x|z) is a 3D renderer, only q(z|x) must be approximated.
    * The model has to learn to count the objects and to estimate per object its identity (class) and pose.
    * They use "finite-differencing" to get gradients through the renderer and use "score function estimators" to get gradients with respect to discrete variables.
    * They first test with a setup where the object count is always 1. The network learns to accurately recover the object parameters.
    * A similar "normal" network has much more problems with recovering the parameters, especially rotation, because the conditional probabilities are multi-modal. The lower bound maximization strategy seems to work better in those cases.
    * In a second experiment with multiple complex objects, AIR also achieves high reconstruction accuracy.
