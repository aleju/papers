# Paper

* **Title**: Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks
* **Authors**: Shaoqing Ren, Kaiming He, Ross Girshick, Jian Sun
* **Link**: https://arxiv.org/abs/1506.01497
* **Tags**: Neural Network, RCNN
* **Year**: 2015

# Summary

* What
  * R-CNN and its successor Fast R-CNN both rely on a "classical" method to find region proposals in images (i.e. "Which regions of the image look like they *might* be objects?").
  * That classical method is selective search.
  * Selective search is quite slow (about two seconds per image) and hence the bottleneck in Fast R-CNN.
  * They replace it with a neural network (region proposal network, aka RPN).
  * The RPN reuses the same features used for the remainder of the Fast R-CNN network, making the region proposal step almost free (about 10ms).

* How
  * They now have three components in their network:
    * A model for feature extraction, called the "feature extraction network" (**FEN**). Initialized with the weights of a pretrained network (e.g. VGG16).
    * A model to use these features and generate region proposals, called the "Region Proposal Network" (**RPN**).
    * A model to use these features and region proposals to classify each regions proposal's object and readjust the bounding box, called the "classification network" (**CN**). Initialized with the weights of a pretrained network (e.g. VGG16).
    * Usually, FEN will contain the convolutional layers of the pretrained model (e.g. VGG16), while CN will contain the fully connected layers.
    * (Note: Only "RPN" really pops up in the paper, the other two remain more or less unnamed. I added the two names to simplify the description.)
    * Rough architecture outline:
      * ![Architecture](images/Faster_R-CNN__architecture.jpg?raw=true "Architecture")
  * The basic method at test is as follows:
    1. Use FEN to convert the image to features.
    2. Apply RPN to the features to generate region proposals.
    3. Use Region of Interest Pooling (RoI-Pooling) to convert the features of each region proposal to a fixed sized vector.
    4. Apply CN to the RoI-vectors to a) predict the class of each object (out of `K` object classes and `1` background class) and b) readjust the bounding box dimensions (top left coordinate, height, width).
  * RPN
    * Basic idea:
      * Place anchor points on the image, all with the same distance to each other (regular grid).
      * Around each anchor point, extract rectangular image areas in various shapes and sizes ("anchor boxes"), e.g. thin/square/wide and small/medium/large rectangles. (More precisely: The features of these areas are extracted.)
      * Visualization:
        * ![Anchor Boxes](images/Faster_R-CNN__anchor_boxes.jpg?raw=true "Anchor Boxes")
      * Feed the features of these areas through a classifier and let it rate/predict the "regionness" of the rectangle in a range between 0 and 1. Values greater than 0.5 mean that the classifier thinks the rectangle might be a bounding box. (CN has to analyze that further.)
      * Feed the features of these areas through a regressor and let it optimize the region size (top left coordinate, height, width). That way you get all kinds of possible bounding box shapes, even though you only use a few base shapes.
    * Implementation:
      * The regular grid of anchor points naturally arises due to the downscaling of the FEN, it doesn't have to be implemented explicitly.
      * The extraction of anchor boxes and classification + regression can be efficiently implemented using convolutions.
        * They first apply a 3x3 convolution on the feature maps. Note that the convolution covers a large image area due to the downscaling.
          * Not so clear, but sounds like they use 256 filters/kernels for that convolution.
        * Then they apply some 1x1 convolutions for the classification and regression.
          * They use `2*k` 1x1 convolutions for classification and `4*k` 1x1 convolutions for regression, where `k` is the number of different shapes of anchor boxes.
      * They use `k=9` anchor box types: Three sizes (small, medium, large), each in three shapes (thin, square, wide).
      * The way they build training examples (below) forces some 1x1 convolutions to react only to some anchor box types.
    * Training:
      * Positive examples are anchor boxes that have an IoU with a ground truth bounding box of 0.7 or more. If no anchor point has such an IoU with a specific box, the one with the highest IoU is used instead.
      * Negative examples are all anchor boxes that have IoU that do not exceed 0.3 for any bounding box.
      * Any anchor point that falls in neither of these groups does not contribute to the loss.
      * Anchor boxes that would violate image boundaries are not used as examples.
      * The loss is similar to the one in Fast R-CNN: A sum consisting of log loss for the classifier and smooth L1 loss (=smoother absolute distance) for regression.
      * Per batch they only sample examples from one image (for efficiency).
      * They use 128 positive examples and 128 negative ones. If they can't come up with 128 positive examples, they add more negative ones.
    * Test:
      * They use non-maximum suppression (NMS) to remove too identical region proposals, i.e. among all region proposals that have an IoU overlap of 0.7 or more, they pick the one that has highest score.
      * They use the 300 proposals with highest score after NMS (or less if there aren't that many).
  * Feature sharing
    * They want to share the features of the FEN between the RPN and the CN.
    * So they need a special training method that fine-tunes all three components while keeping the features extracted by FEN useful for both RPN and CN at the same time (not only for one of them).
    * Their training methods are:
      * Alternating traing: One batch for FEN+RPN, one batch for FEN+CN, then again one batch for FEN+RPN and so on.
      * Approximate joint training: Train one network of FEN+RPN+CN. Merge the gradients of RPN and CN that arrive at FEN via simple summation. This method does not compute a gradient from CN through the RPN's regression task, as that is non-trivial. (This runs 25-50% faster than alternating training, accuracy is mostly the same.)
      * Non-approximate joint training: This would compute the above mentioned missing gradient, but isn't implemented.
      * 4-step alternating training:
        1. Clone FEN to FEN1 and FEN2.
        2. Train the pair FEN1 + RPN.
        3. Train the pair FEN2 + CN using the region proposals from the trained RPN.
        4. Fine-tune the pair FEN2 + RPN. FEN2 is fixed, RPN takes the weights from step 2.
        5. Fine-tune the pair FEN2 + CN. FEN2 is fixed, CN takes the weights from step 3, region proposals come from RPN from step 4.

* Results
  * Example images:
    * ![Example images](images/Faster_R-CNN__examples.jpg?raw=true "Example images")
  * Pascal VOC (with VGG16 as FEN)
    * Using an RPN instead of SS (selective search) slightly improved mAP from 66.9% to 69.9%.
    * Training RPN and CN on the same FEN (sharing FEN's weights) does not worsen the mAP, but instead improves it slightly from 68.5% to 69.9%.
  * Using the RPN instead of SS significantly speeds up the network, from 1830ms/image (less than 0.5fps) to 198ms/image (5fps). (Both stats with VGG16. They also use ZF as the FEN, which puts them at 17fps, but mAP is lower.)
  * Using per anchor point more scales and shapes (ratios) for the anchor boxes improves results.
    * 1 scale, 1 ratio: 65.8% mAP (scale `128*128`, ratio 1:1) or 66.7% mAP (scale `256*256`, same ratio).
    * 3 scales, 3 ratios: 69.9% mAP (scales `128*128`, `256*256`, `512*512`; ratios 1:1, 1:2, 2:1).
  * Two-staged vs one-staged
    * Instead of the two-stage system (first, generate proposals via RPN, then classify them via CN), they try a one-staged system.
    * In the one-staged system they move a sliding window over the computed feature maps and regress at every location the bounding box sizes and classify the box.
    * When doing this, their performance drops from 58.7% to about 54%.


