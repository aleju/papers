# Paper

* **Title**: CornerNet: Detecting Objects as Paired Keypoints
* **Authors**: Hei Law, Jia Deng
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Hei_Law_CornerNet_Detecting_Objects_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Hei_Law_CornerNet_Detecting_Objects_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Object Detection, ECCV 2018
* **Year**: 2018

# Summary

* What
  * Current object detectors follow either a two-stage or single-stage approach.
    * Two-stage: Accurate, but rather slow as they use two different networks to predict RoIs and then classify them.
    * Single-stage
        * Almost as accurate as two-stage detectors nowadays and faster than them.
        * Single-stage detectors place a dense grid of anchor boxes on the image and classify for each of them whether they match an object in the image.
        * There has to be a large number of anchor boxes for different object sizes (and also possible more when the detectors works on multiple scales).
        * This increases memory demands, decreases performance and introduces difficult hyperparameter choices (number of anchors boxes and their sizes).
  * They develop an object detector that is single-stage and free of anchor boxes.
  * Their system predicts bounding boxes by localizing the top left and bottom right corner of an object.
    * They argue that predicting corners is easier than the anchor-based approach.
      There are essentially exactly two points to predict per bounding box,
      while there can be many more valid anchors per object (that are than fine-tuned via regression to fit the object).
  * They also propose a corner pooling layer that complements their technique.
    * Pools along the horizontal/vertical axis.
    * This is useful for objects where no object parts are locally in the corners (e.g. imagine a frontal-view on a human with stretched out arms).
    * They argue that this pooling layer encodes prior knowledge of the prediction task.

* How
  * Corner Detection
    * They predict for each object class two heatmaps: One for top-left corners and one for bottom-right corners.
    * The ground truth heatmaps contain positive values at the locations of corners.
    * Gaussian Ground Truth
      * They argue that corners that are slightly off are still good hits reaching high IoUs.
      * They set the desired target IoU to `t=0.7` and derive from that per corner pair `k` a radius `r_k` describing how far the corners can be off to still fulfill `t`.
      * They place on each ground truth location an unnormalized 2D gaussian `e^(-(x^2 + y^2)/2sigma^2)`, where `sigma=r_k/3`.
    * Corner Position Loss
      * They use a variant of Focal Loss, applied once for the top-left corner heatmaps and once for bottom-right corners.
      * ![comparison](images/CornerNet/corner_loss.jpg?raw=true "comparison")
      * where
        * `y_cij` is ground truth corner heatmap for class `c` at location `y=i` and `x=j`.
        * `p_cij` is the predicted heatmap.
        * `alpha=2, beta=4` are focal loss hyperparameters.
        * `N` is the number of objects in one image.
    * Offsets
      * Predicted heatmaps are often downsampled compared to the ground truth heatmaps.
      * That can lead to predicted locations being slightly off compared to true locations.
      * They compensate for that by letting the network learn that offset (per spatial location).
      * The ground truth for the offset value is:
        * ![comparison](images/CornerNet/corner_offset.jpg?raw=true "comparison")
        * where `n` is the downsampling factor.
      * They apply a smooth L1 loss to the offset predictions.
      * They apply the loss only at the ground truth corner locations.
  * Grouping Corners
    * Top-left and bottom-right corners have to be matched in order to create a full bounding box.
    * They do this by predicting embedding vectors for each top-left and bottom-right corner.
      1. They train these to be similar for corners belonging to the same objects.
      2. They train these to be different for corners belonging to different objects.
    * For (1) they use a pull loss and for (2) a push loss:
      * ![embedding loss](images/CornerNet/embedding_loss.jpg?raw=true "embedding loss")
      * where
        * `e_t_k` is the embedding of the top left corner of object `k`.
        * `e_b_k` is analogously is the embedding of the bottom right corner.
        * `e_k` is the average of `e_t_k` and `e_b_k`.
        * `Delta=1`.
        * Note that there is no ground truth here, because only the distances matter.
    * They apply these losses only at the ground truth corner locations.
  * Corner Pooling
    * In many cases, there is no local evidence for an object around its top-left or bottom-right corners.
    * But there is evidence around its top and left sides (analogously bottom and right sides).
    * Visualization of the problem:
      * ![local evidence](images/CornerNet/local_evidence.jpg?raw=true "local evidence")
    * They compensate for that by max-pooling along the sides of each potential object.
      E.g. for the top-left corner they would max-pool along the corner's row and to the right,
      as well as along the corner's column und to the bottom.
      They sum both of these results.
    * Visualization:
      * ![corner pooling](images/CornerNet/corner_pooling.jpg?raw=true "corner pooling")
  * Architecture
    * They use stacked hourglass networks as their backbone (with minor modifications, e.g. striding instead of max-pooling).
    * They place a corner pooling layer in residual fashion on top of the backbone.
    * They then apply three branches:
      * Corner heatmaps branch (predicts `2*C` channels for `C` classes).
      * Embeddings branch (predicts `?*C` channels).
      * Offsets branch (predicts `2*C` channels).
    * Visualization:
      * ![architecture](images/CornerNet/architecture.jpg?raw=true "architecture")
  * Bounding box extraction
    * To get bounding boxes out of their corner predictions, they first apply non-maximum suppression to their corner heatmaps via a 3x3 max pooling layer.
    * Then they extract the top 100 top-left and bottom-right corners over all classes.
    * They shift them by the predicted offsets.
    * They pair per class the top-left and bottom-right corners with the most similar embeddings, rejecting anything with an L1 distance above 0.5.
    * To these bounding box candidates they apply soft-NMS to remove strongly overlapping bounding boxes.

* Results
  * Loss weightings: They weight their corner heatmaps loss with `1.0`, the offset loss also with `1.0` and the pull and push losses for the embeddings with `0.1` each.
  * They train on 10 PASCAL Titan X.
  * For inference they zero-pad images to the desired input size (511x511) and feed the padded image as well as its horizontally flipped version through the network.
  * They need about 244ms per image for inference.
  * They train and test on COCO.
  * Corner Loss
    * Corner Pooling is essential for the performance of the network.
    * It improves AP by about 2 percentage points.
    * It is especially important for large objects (+3.7 AP), not for small objects (+0.1 AP).
  * Location Penalty (via gaussians)
    * They investigate whether it is necessary to reduce the location penalty in the corner location heatmaps by using gaussians.
    * They compare using
      1. no penalty reduction (just set corner locations to `1`, everything else to `0`),
      2. placing gaussians with a fixed radius of `2.5` and
      3. placing gaussians with object-dependent radii.
    * Option (3) performs best, (1) worst. (2) is in between the two options, usually half-way from (1) to (3).
    * They observe increases of AP between 5 and 6 percentage points when using (3) as opposed to (1).
    * They difference is more pronounced for large objects (about 12 points) as opposed to small objects (2.3 points).
  * Importance of each branch
    * They evaluate which branch (corner location heatmaps, offsets, embeddings) has most influence on the AP.
    * They replace the predicted corner location heatmaps with ground truth heatmaps and increase AP by about 35 points (to 74.0%),
      suggesting that their corner heatmap prediction is the main bottleneck.
    * They then add ground truth offsets and improve by 13.1 points (to 87.1%), suggesting that the offset prediction still has a significant impact on overall AP.
    * This leaves 12.9 points for the other components (embedding prediction, bounding box extraction).
  * Final results
    * They reach 42.1 AP in a multi-scale approach.
      (I guess they feed the images in at multiple scales? Not really explained.
       In previous chapters they write specifically they don't use multi-scale feature maps,
       but only the final feature map.)
      They beat the best multi-scale competitor (RefineDet512) by 0.3 points.
    * They reach 40.5 AP in a single-scale approach. They beat the best single-scale competitor (RetinaNet800) by 1.4 points.
    * Example predictions and extracted bounding boxes (each left: top-left corner, each right: bottom-right):
      * ![predictions](images/CornerNet/predictions.jpg?raw=true "predictions")

