# Paper

* **Title**: Detect to Track and Track to Detect
* **Authors**: Christoph Feichtenhofer, Axel Pinz, Andrew Zisserman
* **Link**: https://arxiv.org/abs/1710.03958
* **Tags**: Neural Network, RCNN, tracking, tubelet, tracklet, viterbi
* **Year**: 2017

# Summary

* What
  * They suggest a variation of Faster R-CNN architectures that also tracks objects in videos
    (additionally to detecting them, i.e. additionally to bounding box detection).
  * The model does detection and tracking in one forward pass.
  * The "tracking" here is rather primitive: It only signals where an object frame `t` will likely be in frame `t+x`.
    That information can then be used to compute chains of bounding boxes over time.

* How
  * Architecture
    * They base their model on R-FCN, which is similar to Faster R-CNN.
    * They have two images/frames (`t` and `t+x`) as inputs.
    * Same as in Faster R-CNN:
      * They apply a base network (e.g. ResNet) to both of them.
      * They use an RPN to locate RoIs in the feature maps.
      * They use RoI-Pooling to extract and pool each RoI's features.
      * They apply classification (object class) and regression (object location/dimensions) to each RoI.
    * They compute correlations between the feature maps of both frames.
    * They then extract and pool each RoI from the feature and correlation maps of from `t`.
    * They then apply a tracking branch, which predicts the new position (x, y) and dimensions (height, width) of the RoI in frame `t+x`.
    * That prediction can be used to find a matching bounding box in frame `t+x`, which again can be used to track objects over time.
    * During training, they use a smooth L1 loss for the tracking branch.
    * Visualization of the architecture:
      * ![architecture](images/Detect_to_Track_and_Track_to_Detect/architecture.jpg?raw=true "architecture")
    * Visualization of the tracking process:
      * ![tracking process](images/Detect_to_Track_and_Track_to_Detect/tracking_process.jpg?raw=true "tracking process")
  * Correlation
    * Correlations are estimated between the feature maps of frame `t` and `t+x`.
    * In the simplest form, correlation is measured by extracting some point (i,j) from the feature maps `t` and `t+x` (resulting in two vectors)
      and then computing the scalar product (resulting in a scalar).
    * They use a more complex correlation, which compares point (i,j) in frame `t` not only to (i,j) in `t+x`, but to (i+d,j+q),
      where d and q come from an interval. I.e. they compare to a neighbourhood in frame `t+x`.
    * They use d=8 (same for q), resulting in 64 correlation values per (i,j).
    * Furthermore, they compute correlations for multiple scales (i.e. early and late in the base network).
      (With striding so that the correlation maps end up having the same sizes.)
  * Tubelets
    * The previous architecture only regresses the new location of a bounding box in frame `t+x`.
    * From that information they derive tubelets, tracks of bounding boxes over multiple frames (i.e. objects tracked over time).
    * Core of the method to do that are **scores** between **pairs of bounding boxes** (between frame `t` and `t+x`).
    * The scores are computed per class, i.e. no score between two bounding boxes of different classes.
    * Each score has three components:
      * (a) Probability of the bounding box in frame `t` having the specified class,
      * (b) Probability of the bounding box in frame `t+x` having the specified class,
      * (c) A flag whether the bounding box in frame `t+x` matches the expected future position of the bounding box in frame `t`.
        The future position is estimated using the new tracking branch.
        The flag has a value of `1` if the IoU between future bounding box and expectation is >0.5.
    * Once all scores are computed, the **Viterbi** algorithm can be used to compute optimal paths over the frames, leading to tracked objects,
      which are the tubelets.
    * After generating a tubelet, they increase the object detection scores of the respective class of all bounding boxes in the tubelet.
      (Because future frames are now giving support that a bounding box really does have a specific object class.)

* Results
  * They train on ImageNet VID dataset (after first training on standard ImageNet).
  * Training with the tracking branch improves raw object detection by about 1.6 points mAP.
  * Predicting on two frames (e.g. `t` and `t+1`) with tracking improves object detection scores for some classes significantly (e.g. +9 points for rabbits).
  * Adding a significant gap between the frames (`t` and `t+10`) is still enough to improve object detection scores (a bit less though).
  * Results overview:
    * ![results](images/Detect_to_Track_and_Track_to_Detect/results.jpg?raw=true "results")
  * Their tracking branch barely adds runtime to the model. Only the tubelet generation adds significant time.
    They modify it (somehow, not described) to a non-causal(?) version, which runs faster.
    Additional runtime per frame with tracking is then 14ms.

