# Paper

* **Title**: A Neural Algorithm of Artistic Style
* **Authors**: Leon A. Gatys, Alexander S. Ecker, Matthias Bethge
* **Link**: http://arxiv.org/abs/1508.06576
* **Tags**: Neural Network, Art, VGG, Gram Matrix, Texture
* **Year**: 2015

# Summary

* What
  * The paper describes a method to separate content and style from each other in an image.
  * The style can then be transfered to a new image.
  * Examples:
    * Let a photograph look like a painting of van Gogh.
    * Improve a dark beach photo by taking the style from a sunny beach photo.

* How
  * They use the pretrained 19-layer VGG net as their base network.
  * They assume that two images are provided: One with the *content*, one with the desired *style*.
  * They feed the content image through the VGG net and extract the activations of the last convolutional layer. These activations are called the *content representation*.
  * They feed the style image through the VGG net and extract the activations of all convolutional layers. They transform each layer to a *Gram Matrix* representation. These Gram Matrices are called the *style representation*.
  * How to calculate a *Gram Matrix*:
    * Take the activations of a layer. That layer will contain some convolution filters (e.g. 128), each one having its own activations.
    * Convert each filter's activations to a (1-dimensional) vector.
    * Pick all pairs of filters. Calculate the scalar product of both filter's vectors.
    * Add the scalar product result as an entry to a matrix of size `#filters x #filters` (e.g. 128x128).
    * Repeat that for every pair to get the Gram Matrix.
    * The Gram Matrix roughly represents the *texture* of the image.
  * Now you have the content representation (activations of a layer) and the style representation (Gram Matrices).
  * Create a new image of the size of the content image. Fill it with random white noise.
  * Feed that image through VGG to get its content representation and style representation. (This step will be repeated many times during the image creation.)
  * Make changes to the new image using gradient descent to optimize a loss function.
    * The loss function has two components:
      * The mean squared error between the new image's content representation and the previously extracted content representation.
      * The mean squared error between the new image's style representation and the previously extracted style representation.
    * Add up both components to get the total loss.
    * Give both components a weight to alter for more/less style matching (at the expense of content matching).


![Examples](images/A_Neural_Algorithm_for_Artistic_Style__examples.jpg?raw=true "Examples")

*One example input image with different styles added to it.*

-------------------------

# Rough chapter-wise notes

* Page 1
  * A painted image can be decomposed in its content and its artistic style.
  * Here they use a neural network to separate content and style from each other (and to apply that style to an existing image).

* Page 2
  * Representations get more abstract as you go deeper in networks, hence they should more resemble the actual content (as opposed to the artistic style).
  * They call the feature responses in higher layers *content representation*.
  * To capture style information, they use a method that was originally designed to capture texture information.
  * They somehow build a feature space on top of the existing one, that is somehow dependent on correlations of features. That leads to a "stationary" (?) and multi-scale representation of the style.

* Page 3
  * They use VGG as their base CNN.

* Page 4
  * Based on the extracted style features, they can generate a new image, which has equal activations in these style features.
  * The new image should match the style (texture, color, localized structures) of the artistic image.
  * The style features become more and more abtstract with higher layers. They call that multi-scale the *style representation*.
  * The key contribution of the paper is a method to separate style and content representation from each other.
  * These representations can then be used to change the style of an existing image (by changing it so that its content representation stays the same, but its style representation matches the artwork).

* Page 6
  * The generated images look most appealing if all features from the style representation are used. (The lower layers tend to reflect small features, the higher layers tend to reflect larger features.)
  * Content and style can't be separated perfectly.
  * Their loss function has two terms, one for content matching and one for style matching.
  * The terms can be increased/decreased to match content or style more.

* Page 8
  * Previous techniques work only on limited or simple domains or used non-parametric approaches (see non-photorealistic rendering).
  * Previously neural networks have been used to classify the time period of paintings (based on their style).
  * They argue that separating content from style might be useful and many other domains (other than transfering style of paintings to images).

* Page 9
  * The style representation is gathered by measuring correlations between activations of neurons.
  * They argue that this is somehow similar to what "complex cells" in the primary visual system (V1) do.
  * They note that deep convnets seem to automatically learn to separate content from style, probably because it is helpful for style-invariant classification.

* Page 9, Methods
  * They use the 19 layer VGG net as their basis.
  * They use only its convolutional layers, not the linear ones.
  * They use average pooling instead of max pooling, as that produced slightly better results.

* Page 10, Methods
  * The information about the image that is contained in layers can be visualized. To do that, extract the features of a layer as the labels, then start with a white noise image and change it via gradient descent until the generated features have minimal distance (MSE) to the extracted features.
  * The build a style representation by calculating Gram Matrices for each layer.

* Page 11, Methods
  * The Gram Matrix is generated in the following way:
    * Convert each filter of a convolutional layer to a 1-dimensional vector.
    * For a pair of filters i, j calculate the value in the Gram Matrix by calculating the scalar product of the two vectors of the filters.
    * Do that for every pair of filters, generating a matrix of size #filters x #filters. That is the Gram Matrix.
  * Again, a white noise image can be changed with gradient descent to match the style of a given image (i.e. minimize MSE between two Gram Matrices).
  * That can be extended to match the style of several layers by measuring the MSE of the Gram Matrices of each layer and giving each layer a weighting.

* Page 12, Methods
  * To transfer the style of a painting to an existing image, proceed as follows:
    * Start with a white noise image.
    * Optimize that image with gradient descent so that it minimizes both the content loss (relative to the image) and the style loss (relative to the painting).
    * Each distance (content, style) can be weighted to have more or less influence on the loss function.
