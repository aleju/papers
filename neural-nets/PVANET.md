# Paper

* **Title**: PVANET: Deep but Lightweight Neural Networks for Real-time Object Detection
* **Authors**: Kye-Hyeon Kim, Sanghoon Hong, Byungseok Roh, Yeongjae Cheon, Minje Park
* **Link**: https://arxiv.org/abs/1608.08021
* **Tags**: Neural Network, RCNN, ResNet, Inception
* **Year**: 2016

# Summary

* What
  * They present a variation of Faster R-CNN.
    * Faster R-CNN is a model that detects bounding boxes in images.
  * Their variation is about as accurate as the best performing versions of Faster R-CNN.
  * Their variation is significantly faster than these variations (roughly 50ms per image).
* How
  * PVANET reuses the standard Faster R-CNN architecture:
    * A base network that transforms an image into a feature map.
    * A region proposal network (RPN) that uses the feature map to predict bounding box candidates.
    * A classifier that uses the feature map and the bounding box candidates to predict the final bounding boxes.
  * PVANET modifies the base network and keeps the RPN and classifier the same.
  * Inception
    * Their base network uses eight Inception modules.
    * They argue that these are good choices here, because they are able to represent an image at different scales (aka at different receptive field sizes)
      due to their mixture of 3x3 and 1x1 convolutions.
      * ![Receptive field sizes in inception modules](images/PVANET__inception_fieldsize.jpg?raw=true "Receptive field sizes in inception modules")
    * Representing an image at different scales is useful here in order to detect both large and small bounding boxes.
    * Inception modules are also reasonably fast.
    * Visualization of their Inception modules:
      * ![Inception modules architecture](images/PVANET__inception_modules.jpg?raw=true "Inception modules architecture")
  * Concatenated ReLUs
    * Before the eight Inception modules, they start the network with eight convolutions using concatenated ReLUs.
    * These CReLUs compute both the classic ReLU result (`max(0, x)`) and concatenate to that the negated result, i.e. something like `f(x) = max(0, x <concat> (-1)*x)`.
    * That is done, because among the early one can often find pairs of convolution filters that are the negated variations of each other.
      So by adding CReLUs, the network does not have to compute these any more, instead they are created (almost) for free, reducing the computation time by up to 50%.
    * Visualization of their final CReLU block:
      * TODO
      * ![CReLU modules](images/PVANET__crelu.jpg?raw=true "CReLU modules")
  * Multi-Scale output
    * Usually one would generate the final feature map simply from the output of the last convolution.
    * They instead combine the outputs of three different convolutions, each resembling a different scale (or level of abstraction).
    * They take one from an early point of the network (downscaled), one from the middle part (kept the same) and one from the end (upscaled).
    * They concatenate these and apply a 1x1 convolution to generate the final output.
  * Other stuff
    * Most of their network uses residual connections (including the Inception modules) to facilitate learning.
    * They pretrain on ILSVRC2012 and then perform fine-tuning on MSCOCO, VOC 2007 and VOC 2012.
    * They use plateau detection for their learning rate, i.e. if a moving average of the loss does not improve any more, they decrease the learning rate. They say that this increases accuracy significantly.
    * The classifier in Faster R-CNN consists of fully connected layers. They compress these via Truncated SVD to speed things up. (That was already part of Fast R-CNN, I think.)
* Results
  * On Pascal VOC 2012 they achieve 82.5% mAP at 46ms/image (Titan X GPU).
    * Faster R-CNN + ResNet-101: 83.8% at 2.2s/image.
    * Faster R-CNN + VGG16: 75.9% at 110ms/image.
    * R-FCN + ResNet-101: 82.0% at 133ms/image.
  * Decreasing the number of region proposals from 300 per image to 50 almost doubles the speed (to 27ms/image) at a small loss of 1.5 percentage points mAP.
  * Using Truncated SVD for the classifier reduces the required timer per image by about 30% at roughly 1 percentage point of mAP loss.

