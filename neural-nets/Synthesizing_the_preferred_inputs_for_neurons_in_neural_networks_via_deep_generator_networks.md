# Paper

* **Title**: Synthesizing the preferred inputs for neurons in neural networks via deep generator networks
* **Authors**: Anh Nguyen, Alexey Dosovitskiy, Jason Yosinski, Thomas Brox, Jeff Clune
* **Link**: http://arxiv.org/abs/1605.09304v3
* **Tags**: Neural Network, generative, GAN
* **Year**: 2016

# Summary

* What
  * They suggest a new method to generate images that maximizes the activations of specific neurons in a trained network.
    * E.g. if your trained network contains a neuron that is active whenever there is a car in an image, the method should generate images containing cars.
    * Such methods can be used to investigate what exactly a network has learned.
  * There are plenty of methods like this one. They usually differ from each other by using different *natural image priors*.
    * A natural image prior is a restriction on the generated images.
    * Such a prior pushes the generated images towards realistic looking ones.
    * Without such a prior it is easy to generate images that lead to high activations of specific neurons, but don't look realistic at all (e.g. they might look psychodelic or like white noise).
      * That's because the space of possible images is extremely high-dimensional and can therefore hardly be covered reliably by a single network. Note also that training datasets usually only show a very limited subset of all possible images.
    * Their work introduces a new natural image prior.

* How
  * Usually, if one wants to generate images that lead to high activations, the basic/naive method is to:
    1. Start with a noise image,
    2. Feed that image through the network,
    3. Compute an error that is high if the activation of the specified neuron is low (analogous for high activation),
    4. Backpropagate the error through the network,
    5. Change the noise image according to the gradient,
    6. Repeat.
  * So, the noise image is basically treated like weights in the network.
  * Their alternative method uses a generator G that transforms noise vectors into images and is created using the method from [Generating Images with Perceptual Similarity Metrics based on Deep Networks](Generating_Images_with_Perceptual_Similarity_Metrics_based_on_Deep_Networks.md).
    * TODO
  * Their modified steps are:
    1. *(New step)* Start with a noise vector,
    2. *(New step)* Feed that vector through G resulting in an image,
    3. *(Same)* Feed that image through the network,
    4. *(Same)* Compute an error that is high if the activation of the specified neuron is low (analogous for high activation),
    5. *(Same)* Backpropagate the error through the network,
    6. *(Modified)* Change the noise *vector* according to the gradient,
    7. *(Same)* Repeat.
  * Additionally they do:
    * Apply an L2 norm to the noise vector, which adds pressure to each component to take low values. They say that this improved the results.
    * Clip each component of the noise vector to a range `[0, a]`, which improved the results significantly.
      * The range starts at `0`, because their Generator is based on ReLUs.
      * `a` is derived from test images and set to 3 standard diviations of the mean activation of that component.

* Results
  * ![Examples](images/Synthesizing_the_preferred_inputs_for_neurons_in_neural_networks_via_deep_generator_networks__examples.jpg?raw=true "Examples")
