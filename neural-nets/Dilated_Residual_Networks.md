# Paper

* **Title**: Dilated Residual Networks
* **Authors**: Fisher Yu, Vladlen Koltun, Thomas Funkhouser
* **Link**: https://arxiv.org/abs/1705.09914
* **Tags**: Neural Network, residual, dilated
* **Year**: 2017

# Summary

* What
  * They change standard ResNets to contain higher resolution feature maps (i.e. more height/width) and to use [dilated convolutions](Multi-Scale_Context_Aggregation_by_Dilated_Convolutions.md).
  * This makes them more useful for e.g. segmentation.
  * They identify a gridding related problem and how to fix it.

* How
  * Changes to ResNet
    * ResNets are organized in five blocks of (each multiple) residual convolutions.
    * They increase the feature map resolutions of the fourth and fifth block (to 2x and 4x).
    * They increase simultanously the dilation of each convolution in the fourth and fifth block (to dilation 2 and 4).
    * This is a common practice and known as the "á trous trick".
  * Degridding
    * When using dilated convolutions one can end up with grid-like patterns in the generated feature maps.
    * This happens when the image has higher-frequency content than the sampling of the dilated convolution.
    * The below image shows an example for a convolution with dilation 2 and equal weights for all 3x3 parameters:
      * ![gridding](images/Dilated_Residual_Networks/gridding.jpg?raw=true "gridding")
    * These grid-like patterns make the network less useful for segmentation tasks.
    * They fix the gridding problems with two steps:
      1. They remove the max pooling at the start of the network and replace it with convolutions.
         This is done, because max pooling led to high-frequency high-amplitude components in their tests.
      2. They add convolutions with less or no dilation after the dilated convolutions (i.e. at the end of the network).
         These "smoothen" the feature maps, thereby fixing the grid-like patterns.
    * They get three networks:
      * **DRN-A-18**: ResNet with 18 layers and dilation (2 in block 4 and 4 in block 5).
      * **DRN-B-26**: Like DRN-A-18, but max pooling is replaced by four residual convolutions (in two blocks, each two convs).
        They also add four residual convolutions at the end of the network (in two blocks, each two convs).
      * **DRN-C-26**: Like DRN-B-26, but the four convolutions at the end of the network are not residual.
    * The motivation to use non-residual convolutions in DRN-C-26 is that residual ones can propagate the problems (grid-like patterns) more easily to the output.
    * Visualization of the architectures:
      * ![architectures](images/Dilated_Residual_Networks/architectures.jpg?raw=true "architectures")
    * Visualization of the effect of using max pooling (DRN-A-XX) and replacing it with convolutions (DRN-B-XX):
      * ![max pooling](images/Dilated_Residual_Networks/maxpooling.jpg?raw=true "max pooling")

* Results
  * Example visualizations of the generated feature maps by each network:
    * ![feature maps](images/Dilated_Residual_Networks/feature_maps.jpg?raw=true "feature maps")
  * ImageNet classification
    * Their networks perform quite a bit better than ResNets of comparable size.
    * The ranking of their networks seems to be: 1. DRN-C-XX, 2. DRN-B-XX, 3. DRN-C-XX.
      The difference between B and C isn't that big, but between A and B it is.
      To a degree this is expected, as B and C have four more convolutions than A (due to max pooling being replaced by convs).
      The effect though seems to be a bit stronger than just that.
    * Their DRN-C-42 network is only a bit less accurate than ResNet-101.
  * Object Localization
    * They suggest a method for bounding box detection without explicit training for that (i.e. when only training on ImageNet for classification).
    * Sounds like they just "try" every possible bounding box within a range of heights/widths at every location in the final feature map.
      Then they pick the one with highest activation, if it is above a threshold.
    * When applying that method to their networks and ResNets they get significantly better results with their networks.
      Not that surprising, considering their final feature map resolutions are four times higher than in ResNet.
  * Semantic Segmentation
    * They apply their models to the Cityscapes dataset.
    * They get a 70.9 mean IoU for DRN-C-42, while the reported best value for ResNet-101 is 66.6.
  * No information regarding runtimes in the paper.
    Dilated ResNets are most likely going to run slower as they work with higher resolution feature maps.
    They will also require more RAM.

