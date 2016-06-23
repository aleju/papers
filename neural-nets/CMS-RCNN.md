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
  * Subparts of their model are:
    * MS-RPN (Multi-Scale Region Proposal Network): "Looks" at the feature maps of a network at multiple scales (i.e. before/after pooling layers) and suggests regions for possible faces.
    * CMS-CNN (Contextual Multi-Scale CNN): "Looks" at feature maps of a network suggested by MS-RPN at multiple scales and classifies whether these regions contains faces. Uses some area around these regions as additional information (suspected regions of bodies).
  * 

* Results

![Architecture](images/Accurate_Image_Super-Resolution__architecture.png?raw=true "Architecture")

*Architecture of the model.*
