# Paper

* **Title**: Combining Markov Random Fields and Convolutional Neural Networks for Image Synthesis
* **Authors**: Chuan Li, Michael Wand
* **Link**: http://arxiv.org/abs/1601.04589
* **Tags**: Neural Network, artistic style, markov random field
* **Year**: 2016

# Summary

* What
  * They describe a method that applies the style of a source image to a target image.
    * Example: Let a normal photo look like a van Gogh painting.
    * Example: Let a normal car look more like a specific luxury car.
  * Their method builds upon the well known [artistic style paper](A_Neural_Algorithm_for_Artistic_Style.md) and uses a new MRF prior.
  * The prior leads to locally more plausible patterns (e.g. less artifacts).

* How
  * They reuse the content loss from the [artistic style paper](A_Neural_Algorithm_for_Artistic_Style.md).
    * The content loss was calculated by feed the source and target image through a network (here: VGG19) and then estimating the squared error of the euclidean distance between one or more hidden layer activations.
    * They use layer `relu4_2` for the distance measurement.
  * They replace the original style loss with a MRF based style loss.
    * Step 1: Extract from the source image `k x k` sized overlapping patches.
    * Step 2: Perform step (1) analogously for the target image.
    * Step 3: Feed the source image patches through a pretrained network (here: VGG19) and select the representations `r_s` from specific hidden layers (here: `relu3_1`, `relu4_1`).
    * Step 4: Perform step (3) analogously for the target image. (Result: `r_t`)
    * Step 5: For each patch of `r_s` find the best matching patch in `r_t` (based on normalized cross correlation).
    * Step 6: Calculate the sum of squared errors (based on euclidean distances) of each patch in `r_s` and its best match (according to step 5).
  * They add a regularizer loss.
    * The loss encourages smooth transitions in the synthesized image (i.e. few edges, corners).
    * It is based on the raw pixel values of the last synthesized image.
    * For each pixel in the synthesized image, they calculate the squared x-gradient and the squared y-gradient and then add both.
    * They use the sum of all those values as their loss (i.e. `regularizer loss = <sum over all pixels> x-gradient^2 + y-gradient^2`).
  * Their whole optimization problem is then roughly `image = argmin_image MRF-style-loss + alpha1 * content-loss + alpha2 * regularizer-loss`.
  * In practice, they start their synthesis with a low resolution image and then progressively increase the resolution (each time performing some iterations of optimization).
  * In practice, they sample patches from the style image under several different rotations and scalings.

* Results
  * In comparison to the original artistic style paper:
    * Less artifacts.
    * Their method tends to preserve style better, but content worse.
    * Can handle photorealistic style transfer better, so long as the images are similar enough. If no good matches between patches can be found, their method performs worse.

![Non-photorealistic example images](images/Combining_MRFs_and_CNNs_for_Image_Synthesis__examples.png?raw=true "Non-photorealistic example images")

*Non-photorealistic example images. Their method vs. the one from the original artistic style paper.*


![Photorealistic example images](images/Combining_MRFs_and_CNNs_for_Image_Synthesis__examples_real.png?raw=true "Photorealistic example images")

*Photorealistic example images. Their method vs. the one from the original artistic style paper.*
