# Paper

* **Title**: Semantic Style Transfer and Turning Two-Bit Doodles into Fine Artwork
* **Authors**: Alex J. Champandard
* **Link**: http://arxiv.org/abs/1603.01768
* **Tags**: Neural Network, artistic style, markov random field
* **Year**: 2016

# Summary

* What
  * They describe a method to transfer image styles based on semantic classes.
  * This allows to:
    * (1) Transfer styles between images more accurately than with previous models. E.g. so that the background of an image does not receive the style of skin/hair/clothes/... seen in the style image. Skin in the synthesized image should receive the style of skin from the style image. Same for hair, clothes, etc.
    * (2) Turn simple doodles into artwork by treating the simplified areas in the doodle as semantic classes and annotating an artwork with these same semantic classes. (E.g. "this blob should receive the style from these trees.")

* How
  * Their method is based on [Combining Markov Random Fields and Convolutional Neural Networks for Image Synthesis](Combining_MRFs_and_CNNs_for_Image_Synthesis.md).
  * They use the same content loss and mostly the same MRF-based style loss. (Apparently they don't use the regularization loss.)
  * They change the input of the MRF-based style loss.
    * Usually that input would only be the activations of a VGG-layer (for the synthesized image or the style source image).
    * They add a semantic map with weighting `gamma` to the activation, i.e. `<representation of image> = <activation of specific layer for that image> || gamma * <semantic map>`.
    * The semantic map has N channels with 1s in a channel where a specific class is located (e.g. skin).
    * The semantic map has to be created by the user for both the content image and the style image.
    * As usually for the MRF loss, patches are then sampled from the representations. The semantic maps then influence the distance measure. I.e. patches are more likely to be sampled from the same semantic class.
    * Higher `gamma` values make it more likely to sample from the same semantic class (because the distance from patches from different classes gets larger).
  * One can create a small doodle with few colors, then use the colors as the semantic map. Then add a semantic map to an artwork and run the algorithm to transform the doodle into an artwork. 

* Results
  * More control over the transfered styles than previously.
  * Less sensitive to the style weighting, because of the additional `gamma` hyperparameter.
  * Easy transformation from doodle to artwork.

![Example](images/Neural_Doodle__example.png?raw=true "Example")

*Turning a doodle into an artwork. Note that the doodle input image is also used as the semantic map of the input.*
