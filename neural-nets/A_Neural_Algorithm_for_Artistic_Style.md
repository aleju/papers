# Paper

**Title**: A Neural Algorithm of Artistic Style
**Authors**: Leon A. Gatys, Alexander S. Ecker, Matthias Bethge
**Link**: http://arxiv.org/abs/1508.06576
**Tags**: Neural Network, Art, VGG, Gram Matrix, Texture
**Year**: 2015

# Summary


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
  * The information about the image that is contained in layers can be visualized. To do that, extract the features of a layer as the labels, then start with a white noise image and change it via gradient descendt until the generated features have minimal distance (MSE) to the extracted features.
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
    * Optimize that image with gradient descendt so that it minimizes both the content loss (relative to the image) and the style loss (relative to the painting).
    * Each distance (content, style) can be weighted to have more or less influence on the loss function.
