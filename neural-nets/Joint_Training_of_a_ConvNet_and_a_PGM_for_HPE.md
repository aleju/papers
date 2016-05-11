# Paper

* **Title**: Joint Training of a Convolutional Network and a Graphical Model for Human Pose Estimation
* **Authors**: Jonathan Tompson, Arjun Jain, Yann LeCun, Christoph Bregler
* **Link**: http://arxiv.org/abs/1406.2984
* **Tags**: Neural Network, pgm, human pose estimation
* **Year**: 2014

# Summary

![Examples](images/A_Neural_Algorithm_for_Artistic_Style__examples.jpg?raw=true "Examples")

*One example input image with different styles added to it.*

-------------------------

# Rough chapter-wise notes

* (1) Introduction
  * Human Pose Estimation (HPE) from RGB images is difficult due to the high dimensionality of the input.
  * Approaches:
    * Deformable-part models: Traditionally based on hand-crafted features.
    * Deep-learning based disciminative models: Recently outperformed other models. However, it is hard to incorporate priors (e.g. possible joint- inter-connectivity) into the model.
  * They combine:
    * A part-detector (ConvNet, utilizes multi-resolution feature representation with overlapping receptive fields)
    * Part-based Spatial-Model (approximates loopy belief propagation)
  * They backpropagate through the spatial model and then the part-detector.

* (3) Model
  * (3.1) Convolutional Network Part-Detector
    * This model locates possible positions of human key joints in the image ("part detector").
    * Input: RGB image.
    * Output: 4 heatmaps, one per key joint (per pixel: likelihood).
    * They use a fully convolutional network.
    * They argue that applying convolutions to every pixel is similar to moving a sliding window over the image.
    * They use two receptive field sizes for their "sliding window": A large but coarse/blurry one, a small but fine one.
    * To implement that, they use two branches. Both branches are mostly identical (convolutions, poolings, ReLU). They simply feed a downscaled (half width/height) version of the input image into the coarser branch. At the end they upscale the coarser branch once and then merge both branches. 
    * After the merge they apply 9x9 convolutions and then 1x1 convolutions to get it down to 4xHxW (H=60, W=90 where expected input was H=320, W=240).
  * (3.2) Higher-level Spatial-Model
    * This model takes the detected joint positions (heatmaps) and tries to remove those that are probably false positives.
    * 
  * (3.3) Unified Models
    * They combine the part-based model and the spatial model to a single one.
    * They first train only the part-based model, then only the spatial model, then both.

* (4) Results
  * Used datasets: FLIC (4k training images, 1k test, mostly front-facing and standing poses), extended-LSP (10k, 1k).
  * FLIC contains images showing multiple persons with only one being annotated. So for FLIC they add a heatmap of the annotated body torso to the input (i.e. the part-detector does not have to search for the person any more).
  * 
