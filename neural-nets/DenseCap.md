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
      * Each region is translated to a sentence.
      * The region is fed into an LSTM (after a linear layer + ReLU), followed by a special START token.
      * The LSTM outputs multiple words as one-hot-vectors, where each vector has the length `V+1` (i.e. vocabulary size + END token).
      * Loss function is average crossentropy between output words and target words.
      * During test time, words are sampled until an END tag is generated.
  * Loss function
    * Their full loss function has five components:
      * Binary logistic regression for the loss function of the region confidences of the localization layer.
      * Binary logistic regression for the loss function of the region confidences of the recognition layer.
      * Smooth L1 loss on the region predictions of the localization layer.
      * Smooth L1 loss on the region predictions of the recognition layer.
      * Cross-entropy at every time-step of the language model.
    * The language model term has a weight of 1.0, all other components have a weight of 0.1.
  * Training an optimization
    * Initialization: CNN pretrained on ImageNet, all other weights from `N(0, 0.01)`.
    * SGD for the CNN (lr=?, momentum=0.9)
    * Adam everywhere else (lr=1e-6, beta1=0.9, beta2=0.99)
    * CNN is trained after epoch 1. CNN's first four layers are not trained.
    * Batch size is 1.
    * Image size is 720 on the longest side.
    * They use Torch.
    * 3 days of training time.

* (4) Experiments
  * They use the Visual Genome Dataset (94k images, 4.1M regions with captions)
  * Their total vocabulary size is 10,497 words. (Rare words in captions were replaced with `<UNK>`.)
  * They throw away annotations with too many words as well as images with too few/too many regions.
  * They merge heavily overlapping regions to single regions with multiple captions.
  * Dense Captioning
    * Dense captioning task: The model receives one image and produces a set of regions, each having a caption and a confidence score.
    * Evaluation metrics
      * Evaluation of the output is non-trivial.
      * They compare predicted regions with regions from the annotation that have high overlap (above a threshold).
      * They then compare the predicted caption with the captions having similar METEOR score (above a threshold).
      * Instead of setting one threshold for each comparison they use multiple thresholds. Then they calculate the Mean Average Precision using the various pairs of thresholds.
    * Baseline models
      * *TODO*
    * Discrepancy between region and image level statistics
      * *TODO*
    * RPN outperforms external region proposals
      * *TODO*
    * Our model outperforms individual region description
      * *TODO*
    * Qualitative results
      * Finds plenty of good regions and generates reasonable captions for them.
      * Sometimes finds the same region twice.
    * Runtime evaluation
      * 240ms on 720x600 image with 300 region proposals.
      * 166ms on 720x600 image with 100 region proposals.
      * Recognition of region proposals takes up most time.
      * Generating region proposals takes up the 2nd most time.
      * Generating captions for regions (RNN) takes almost no time.
  * Image Retrieval using Regions and Captions
    * 

