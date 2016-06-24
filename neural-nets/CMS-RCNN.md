# Paper

* **Title**: CMS-RCNN: Contextual Multi-Scale Region-based CNN for Unconstrained Face Detection
* **Authors**: Chenchen Zhu, Yutong Zheng, Khoa Luu, Marios Savvides
* **Link**: http://arxiv.org/abs/1606.05413v1
* **Tags**: Neural Network, face
* **Year**: 2016

# Summary

* What
  * They describe a model to locate faces in images.
  * Their model uses information from suspected face regions *and* from the corresponding suspected body regions to classify whether a region contains a face.
  * The intuition is, that seeing the region around the face (specifically where the body should be) can help in estimating whether a suspected face is really a face (e.g. it might also be part of a painting, statue or doll).

* How
  * Their whole model is called "CMS-RCNN" (Contextual Multi-Scale Region-CNN).
  * It is based on the "Faster R-CNN" architecture.
  * It uses the VGG network.
  * Subparts of their model are: MS-RPN, CMS-CNN.
  * MS-RPN finds candidate face regions. CMS-CNN refines their bounding boxes and classifies them (face / not face).
  * **MS-RPN** (Multi-Scale Region Proposal Network)
    * "Looks" at the feature maps of the network (VGG) at multiple scales (i.e. before/after pooling layers) and suggests regions for possible faces.
    * Steps:
      * Feed an image through the VGG network.
      * Extract the feature maps of the three last convolutions that are before a pooling layer.
      * Pool these feature maps so that they have the same heights and widths.
      * Apply L2 normalization to each feature map so that they all have the same scale.
      * Apply a 1x1 convolution to merge them to one feature map.
      * Regress face bounding boxes from that feature map according to the Faster R-CNN technique.
  * **CMS-CNN** (Contextual Multi-Scale CNN):
    * "Looks" at feature maps of face candidates found by MS-RPN and classifies whether these regions contains faces.
    * It also uses the same multi-scale technique (i.e. take feature maps from convs before pooling layers).
    * It uses some area around these face regions as additional information (suspected regions of bodies).
    * Steps:
      * Receive face candidate regions from MS-RPN.
      * Do per candidate region:
        * Calculate the suspected coordinates of the body (only based on the x/y-position and size of the face region, i.e. not learned).
        * Extract the feature maps of the *face* region (at multiple scales) and apply RoI-Pooling to it (i.e. convert to a fixed height and width).
        * Extract the feature maps of the *body* region (at multiple scales) and apply RoI-Pooling to it (i.e. convert to a fixed height and width).
        * L2-normalize each feature map.
        * Concatenate the (RoI-pooled and normalized) feature maps of the face (at multiple scales) with each other (creates one tensor).
        * Concatenate the (RoI-pooled and normalized) feature maps of the body (at multiple scales) with each other (creates another tensor).
        * Apply a 1x1 convolution to the face tensor.
        * Apply a 1x1 convolution to the body tensor.
        * Apply two fully connected layers to the face tensor, creating a vector.
        * Apply two fully connected layers to the body tensor, creating a vector.
        * Concatenate both vectors.
        * Based on that vector, make a classification of whether it is really a face.
        * Based on that vector, make a regression of the face's final bounding box coordinates and dimensions.
  * Note: They use in both networks the multi-scale approach in order to be able to find small or tiny faces. Otherwise, after pooling these small faces would be hard or impossible to detect.

* Results
  * Adding context to the classification (i.e. the body regions) empirically improves the results.
  * Their model achieves the highest recall rate on FDDB compared to other models. However, it has lower recall if only very few false positives are accepted.
  * FDDB ROC curves (theirs is bold red):
    * ![FDDB results](images/CMS-RCNN__fddb.jpg?raw=true "FDDB results")
  * Example results on FDDB:
    * ![FDDB examples](images/CMS-RCNN__examples.jpg?raw=true "FDDB examples")

