# Paper

* **Title**: DenseCap: Fully Convolutional Localization Networks for Dense Captioning
* **Authors**: Justin Johnson, Andrej Karpathy, Li Fei-Fei
* **Link**: http://arxiv.org/abs/1511.07571
* **Tags**: Neural Network, localization, captioning, dense captioning, detection, LSTM, R-CNN
* **Year**: 2015

# Summary

![Reconstructed images](images/DenseCap__reconstructed.png?raw=true "Reconstructed images")

*Lorem ipsum.*



--------------------

# Rough chapter-wise notes

* (1) Introduction
  * They define 4 subtasks of visual scene understanding:
    * Classification: Assign a single label to a whole image
    * Captioning: Assign a sequence of words (description) to a whole image
    * Detection: Find objects in an image and assign a single label to each one
    * Dense Captioning: Find objects in an image and assign a sequence of words (description) to each one
  * They developed a model for dense captioning.
  * It has two three important components:
    * A convoltional network for scene understanding
    * A localization layer for region level predictions. It predicts regions of interest and then uses bilinear sampling to extract the activations of these regions.
    * A recurrent network as the language model
  * They evaluate the model on the large-scale Visual Genome dataset (94k images, 4.1M region captions).

* (3) Model
  * Model architecture
    * Convolutional Network
      * They use VGG-16, but remove the last pooling layer.
      * For an image of size W, H the output is 512xW/16xH/16.
      * That output is the input into the localization layer.
    * Fully Convolutional Localization Layer
      * Input to this layer: Activations from the convolutional network.
      * Output of this layer: Regions of interest, as fixed-sized representations.
        * For B Regions:
          * Coordinates of the bounding boxes (matrix of shape Bx4)
          * Confidence scores (vector of length B)
          * Features (matrix of shape BxCxXxY)
      * Method: Faster R-CNN (pooling replaced by bilinear interpolation)
      * This layer is fully differentiable.
      * The localization layer predicts boxes at anchor points.
      * At each anchor points it proposes `k` boxes using a small convolutional network. It assigns a confidence score and coordinates (center x, center y, height, width) to each proposal.
      * For an image with size 720x540 and k=12 the model would have to predict 17,280 boxes, hence subsampling is used.
      * During training they use minibatches with 256/2 positive and 256/2 negative region examples. A box counts as a positive example for a specific image if it has high overlap (intersection) with an annotated box for that image.
      * During test time they use greedy non-maximum suppression (NMS) (?) to subsample the 300 most confident boxes.
      * The region proposals have varying box sizes, but the output of the localization layer (which will be fed into the RNN) is ought to have fixed sizes.
      * So they project each proposed region onto a fixed sized region. They use bilinear sampling for that projection, which is differentiable.
    * Recognition network
      * Each region is flattened to a one-dimensional vector.
      * That vector is fed through 2 fully connected layers (unknown size, ReLU, dropout), ending with a 4096 neuron layer.
      * The confidence score and box coordinates are also adjusted by the network during that process (fine tuning).
    * RNN Language Model
      * 

