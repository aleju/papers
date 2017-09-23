# Paper

* **Title**: MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications
* **Authors**: Andrew G. Howard, Menglong Zhu, Bo Chen, Dmitry Kalenichenko, Weijun Wang, Tobias Weyand, Marco Andreetto, Hartwig Adam
* **Link**: https://arxiv.org/abs/1704.04861
* **Tags**: Neural Network
* **Year**: 2017

# Summary

* What
  * They suggest a factorization of standard 3x3 convolutions that is more efficient.
  * They build a model based on that factorization. The model has hyperparameters to choose higher performance or higher accuracy.

* How
  * Factorization
    * They factorize the standard 3x3 convolution into one depthwise 3x3 convolution, followed by a pointwise convoluton.
    * Normal 3x3 convolution:
      * Computes per filter and location a weighted average over all filters.
      * For kernel height `kH`, width `kW` and number of input filters/planes `Fin`, it requires `kH*kW*Fin` computations per location.
    * Depthwise 3x3 convolution:
      * Computes per filter and location a weighted average over *one* input filter. E.g. the 13th filter would only computed weighted averages over the 13th input filter/plane and ignore all the other input filters/planes.
      * This requires `kH*kW*1` computations per location, i.e. drastically less than a normal convolution.
    * Pointwise convolution:
      * This is just another name for a normal 1x1 convolution.
      * This is placed after a depthwise convolution in order to compensate the fact that every (depthwise) filter only sees a single input plane.
      * As the kernel size is `1`, this is rather fast to compute.
    * Visualization of normal vs factorized convolution:
      * ![architecture](images/MobileNets/architecture.jpg?raw=true "architecture")
  * Models
    * They use two hyperparameters for their models.
    * `alpha`: Multiplier for the width in the range `(0, 1]`. A value of 0.5 means that every layer has half as many filters.
    * `roh`: Multiplier for the resolution. In practice this is simply the input image size, having a value of `{224, 192, 160, 128}`.

* Results
  * ImageNet
    * Compared to VGG16, they achieve 1 percentage point less accuracy, while using only about 4% of VGG's multiply and additions (mult-adds) and while using only about 3% of the parameters.
    * Compared to GoogleNet, they achieve about 1 percentage point more accuracy, while using only about 36% of the mult-adds and 61% of the parameters.
    * Note that they don't compare to ResNet.
    * Results for architecture choices vs. accuracy on ImageNet:
      * ![results imagenet](images/MobileNets/results_imagenet.jpg?raw=true "results imagenet")
    * Relation between mult-adds and accuracy on ImageNet:
      * ![mult-adds vs accuracy](images/MobileNets/mult-adds_vs_accuracy.jpg?raw=true "mult-adds vs accuracy")
  * Object Detection
    * Their mAP is a bit on COCO when combining MobileNet with SSD (as opposed to using VGG or Inception v2).
    * Their mAP is quite a bit worse on COCO when combining MobileNet with Faster R-CNN.
  * Reducing the number of filters (`alpha`) influences the results more than reducing the input image resolution (`roh`).
  * Making the models shallower influences the results more than making them thinner.

