# Paper

* **Title**: Pixel Recurrent Neural Networks
* **Authors**: Aaron van den Oord, Nal Kalchbrenner, Koray Kavukcuoglu
* **Link**: http://arxiv.org/abs/1601.06759
* **Tags**: Neural Network, recurrent, generative model, LSTM
* **Year**: 2016

# Summary

*Note*: This paper felt rather hard to read. The summary might not have hit exactly what the authors tried to explain.

* What
  * The authors describe multiple architectures that can model the distributions of images.
  * These networks can be used to generate new images or to complete existing ones.
  * The networks are mostly based on RNNs.

* How
  * They define three architectures:
    * Row LSTM:
      * Predicts a pixel value based on all previous pixels in the image.
      * It applies 1D convolutions (with kernel size 3) to the current and previous rows of the image.
      * It uses the convolution results as features to predict a pixel value.
    * Diagonal BiLSTM:
      * Predicts a pixel value based on all previous pixels in the image.
      * Instead of applying convolutions in a row-wise fashion, they apply them to the diagonals towards the top left and top right of the pixel.
      * Diagonal convolutions can be applied by padding the n-th row with `n-1` pixels from the left (diagonal towards top left) or from the right (diagonal towards the top right), then apply a 3x1 column convolution.
    * PixelCNN:
      * Applies convolutions to the region around a pixel to predict its values.
      * Uses masks to zero out pixels that follow after the target pixel.
      * They use no pooling layers.
      * While for the LSTMs each pixel is conditioned on all previous pixels, the dependency range of the CNN is bounded.
  * They use up to 12 LSTM layers.
  * They use residual connections between their LSTM layers.
  * All architectures predict pixel values as a softmax over 255 distinct values (per channel). According to the authors that leads to better results than just using one continuous output (i.e. sigmoid) per channel.
  * They also try a multi-scale approach: First, one network generates a small image. Then a second networks generates the full scale image while being conditioned on the small image.

* Results
  * The softmax layers learn reasonable distributions. E.g. neighboring colors end up with similar probabilities. Values 0 and 255 tend to have higher probabilities than others, especially for the very first pixel.
  * In the 12-layer LSTM row model, residual and skip connections seem to have roughly the same effect on the network's results. Using both yields a tiny improvement over just using one of the techniques alone.
  * They achieve a slightly better result on MNIST than DRAW did.
  * Their negative log likelihood results for CIFAR-10 improve upon previous models. The diagonal BiLSTM model performs best, followed by the row LSTM model, followed by PixelCNN.
  * Their generated images for CIFAR-10 and Imagenet capture real local spatial dependencies. The multi-scale model produces better looking results. The images do not appear blurry. Overall they still look very unreal.


![Generated ImageNet images](images/Pixel_Recurrent_Neural_Networks__imagenet_multiscale.png?raw=true "Generated ImageNet images")

*Generated ImageNet 64x64 images.*


![Image completion](images/Pixel_Recurrent_Neural_Networks__occlusion.png?raw=true "Image completion")

*Completing partially occluded images.*
