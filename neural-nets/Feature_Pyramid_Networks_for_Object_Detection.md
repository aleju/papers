# Paper

* **Title**: Feature Pyramid Networks for Object Detection
* **Authors**: Tsung-Yi Lin, Piotr Dollár, Ross Girshick, Kaiming He, Bharath Hariharan, Serge Belongie
* **Link**: https://arxiv.org/abs/1612.03144
* **Tags**: Neural Network, RCNN
* **Year**: 2016

# Summary

* What
  * They suggest a modified network architecture for object detectors (i.e. bounding box detectors).
  * The architecture aggregates features from many scales (i.e. before each pooling layer) to detect both small and large object.
  * The network is shaped similar to an hourglass.

* How
  * Architecture
    * They have two branches.
    * The first one is similar to any normal network:
      Convolutions and pooling.
      The exact choice of convolutions (e.g. how many) and pooling is determined by the used base network (e.g. ~50 convolutions with ~5x pooling in ResNet-50).
    * The second branch starts at the first one's output.
      It uses nearest neighbour upsampling to re-increase the resolution back to the original one.
      It does not contain convolutions.
      All layers have 256 channels.
    * There are connections between the layers of the first and second branch.
      These connections are simply 1x1 convolutions followed by an addition (similar to residual connections).
      Only layers with similar height and width are connected.
    * Visualization:
      * ![architecture](images/Feature_Pyramid_Networks_for_Object_Detection/architecture.jpg?raw=true "architecture")
  * Integration with Faster R-CNN
    * They base the RPN on their second branch.
    * While usually an RPN is applied to a single feature map of one scale, in their case it is applied to many feature maps of varying scales.
    * The RPN uses the same parameters for all scales.
    * They use anchor boxes, but only of different aspect ratios, not of different scales (as scales are already covered by their feature map heights/widths).
    * Ground truth bounding boxes are associated with the best matching anchor box (i.e. one box among all scales).
    * Everything else is the same as in Faster R-CNN.
  * Integration with Fast R-CNN
    * Fast R-CNN does not use an RPN, but instead usually uses Selective Search to find region proposals (and applies RoI-Pooling to them).
    * Here, they simply RoI-Pool from the FPN's output of the second branch.
    * They do not pool over all scales. Instead they pick only the scale/layer that matches the region proposal's size (based on its height/width).
    * They process each pooled RoI using two 1024-dimensional fully connected layers (initalizes randomly).
    * Everything else is the same as in Fast R-CNN.

* Results
  * Faster R-CNN
    * FPN improves recall on COCO by about 8 points, compared to using standard RPN.
    * Improvement is stronger for small objects (about 12 points).
    * For some reason no AP values here, only recall.
    * The RPN uses some convolutions to transform each feature map into region proposals.
      Sharing the features of these convolutions marginally improves results.
  * Fast R-CNN
    * FPN improves AP on COCO by about 2 points.
    * Improvement is stronger for small objects (about 2.1 points).
