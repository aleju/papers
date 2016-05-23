# Paper

* **Title**: Texture Synthesis Through Convolutional Neural Networks and Spectrum Constraints
* **Authors**: Gang Liu, Yann Gousseau, Gui-Song Xia
* **Link**: http://arxiv.org/abs/1605.01141v3
* **Tags**: Neural Network, artistic style, fourier, texture
* **Year**: 2016

# Summary

* What
  * The well known method of [Artistic Style Transfer](A_Neural_Algorithm_for_Artistic_Style.md) can be used to generate new texture images (from an existing example) by skipping the content loss and only using the style loss.
  * The method however can have problems with large scale structures and quasi-periodic patterns.
  * They add a new loss based on the spectrum of the images (synthesized image and style image), which decreases these problems and handles especially periodic patterns well.

* How
  * Everything is handled in the same way as in the Artistic Style Transfer paper (without content loss).
  * On top of that they add their spectrum loss:
    * The loss is based on a squared distance, i.e. `1/2 d(I_s, I_t)^2`.
      * `I_s` is the last synthesized image.
      * `I_t` is the texture example.
    * `d(I_s, I_t)` then does the following:
      * It assumes that `I_t` is an example for a space of target images.
      * Within that set it finds the image `I_p` which is most similar to `I_s`. That is done using a projection via Fourier Transformations. (See formula 5 in the paper.)
      * The returned distance is then `I_s - I_p`.

* Results
  * Equal quality for textures without quasi-periodic structures.
  * Significantly better quality for textures with quasi-periodic structures.


![Overview](images/Texture_Synthesis_Through_CNNs_and_Spectrum_Constraints__overview.png?raw=true "Overview")

*Overview over their method, i.e. generated textures using style and/or spectrum-based loss.*
