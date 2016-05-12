# Paper

* **Title**: Accurate Image Super-Resolution Using Very Deep Convolutional Networks
* **Authors**: Jiwon Kim, Jung Kwon Lee, Kyoung Mu Lee
* **Link**: http://arxiv.org/abs/1511.04587
* **Tags**: Neural Network, superresolution, residual
* **Year**: 2015

# Summary

* What
  * They describe a model that upscales low resolution images to their high resolution equivalents ("Single Image Super Resolution").
  * Their model uses a deeper architecture than previous models and has a residual component.

* How
  * Their model is a fully convolutional neural network.
  * Input of the model: The image to upscale, *already upscaled to the desired size* (but still blurry).
  * Output of the model: The upscaled image (without the blurriness).
  * They use 20 layers of padded 3x3 convolutions with size 64xHxW with ReLU activations. (No pooling.)
  * They have a residual component, i.e. the model only learns and outputs the *change* that has to be applied/added to the blurry input image (instead of outputting the full image). That change is applied to the blurry input image before using the loss function on it. (Note that this is a bit different from the currently used "residual learning".)
  * They use a MSE between the "correct" upscaling and the generated upscaled image (input image + residual).
  * They use SGD starting with a learning rate of 0.1 and decay it 3 times by a factor of 10.
  * They use weight decay of 0.0001.
  * During training they use a special gradient clipping adapted to the learning rate. Usually gradient clipping restricts the gradient values to `[-t, t]` (`t` is a hyperparameter). Their gradient clipping restricts the values to `[-t/lr, t/lr]` (where `lr` is the learning rate).
  * They argue that their special gradient clipping allows the use of significantly higher learning rates.
  * They train their model on multiple scales, e.g. 2x, 3x, 4x upscaling. (Not really clear how. They probably feed their upscaled image again into the network or something like that?)

* Results
  * Higher accuracy upscaling than all previous methods.
  * Can handle well upscaling factors above 2x.
  * Residual network learns significantly faster than non-residual network.

![Architecture](images/Accurate_Image_Super-Resolution__architecture.png?raw=true "Architecture")

*Architecture of the model.*


![Examples](images/Accurate_Image_Super-Resolution__examples.png?raw=true "Examples")

*Super-resolution quality of their model (top, bottom is a competing model).*

