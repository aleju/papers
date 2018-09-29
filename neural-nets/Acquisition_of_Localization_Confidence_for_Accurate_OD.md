# Paper

* **Title**: Acquisition of Localization Confidence for Accurate Object Detection
* **Authors**: Borui Jiang, Ruixuan Luo, Jiayuan Mao, Tete Xiao, Yuning Jiang
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Borui_Jiang_Acquisition_of_Localization_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Borui_Jiang_Acquisition_of_Localization_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Object Detection, ECCV 2018
* **Year**: 2018

# Summary

* What
  * Current object detectors predict usually a set of bounding boxes, each having a classification score ("how likely it is to be of an object class")
    and regressed bounding box dimensions ("where is the box and how high/wide is it").
  * These results are then filtered through NMS, suppressing all bounding boxes that overlap strongly with another box that has higher classification score.
  * Sometimes the regressed dimensions are also iteratively refined (e.g. in two-stage detectors one can argue that there is one refinement after the RPN).
  * The authors observe that this structure is problematic, as there is no uncertainty-related information regarding the regressed dimensions.
    * For instance in NMS, the suppression works based on classification scores,
      but e.g. a too large bounding box might get a higher classification score than one with optimal fit,
      due to including more context and thereby making the classification more certain.
    * The classification scores are also very non-monotonic with respect to the bounding box dimensions:
      Increasing the bounding box size might first lead to higher classification scores, then lower ones, then higher again.
      * This makes iterative refinement hard or impossible.
  * They propose IoU-Net, which predicts for each bounding box the expected IoU with the ground truth.
    * That predicted IoU is then used in NMS as a replacement for the classification score,
      thereby selecting for the bounding boxes that the model thinks have highest IoU.
    * The predicted IoU can also be used to perform iterative refinement, as it is more monotonic than the classification score.
  * They propose "Precise RoI Pooling" (PrRoI Pooling), which is a version of RoI-Pooling/RoI-Align that does not suffer from quantization inaccuracies.

* How
  * Predicting IoUs
    * They feed each RoI's features through two fully connected layers to regress an IoU value for that RoI.
    * They use one such branch per class.
    * During training they augment the ground truth bounding boxes to generate examples with lower IoUs.
    * Visualization:
      * ![architecture](images/Acquisition_of_Localization_Confidence_for_Accurate_OD/architecture.jpg?raw=true "architecture")
    * Relationship between predicted IoUs and real IoUs (right) vs. classification scores and real IoUs (left):
      * ![predictions vs ious](images/Acquisition_of_Localization_Confidence_for_Accurate_OD/predictions-vs-ious.jpg?raw=true "predictions vs ious")
  * IoU-guided NMS
    * This is essentially the same as classical NMS.
    * They use the predicted IoU values to estimate which box to keep if two are sufficiently overlapping.
    * When one box is suppressed, the remaining box's class score is updated to `max(s_1, s_2)`,
      where `s_1` and `s_2` are the class scores of the two boxes.
  * Optimization-based refinement of bounding boxes
    * The branch to predict IoUs is differentiable.
    * They use this to iteratively improve predicted bounding boxes so that the predicted IoU is maximized.
      * I.e. they backpropagate to the bounding box coordinates and change them according to the gradient.
      * They scale up the gradients according to the bounding box size on the given axis, which is similar to optimizing in log-space.
    * They add some early stopping criteria to not execute this indefinitely.
  * Precise RoI Pooling
    * During bounding box refinement they use a more accurate (quantization free) RoI-Pooling.
    * They use bilinear sampling to approximate the feature value of any continuous location within the feature map.
    * Then they use integration to calculate the pooled value (makes it sound like they always use global pooling?).
    * So while this method computes the integrals over all possible values within the given range, RoI Align only samples at N=4 locations per side.
    * Visualization of the differences:
      * ![prroi pooling](images/Acquisition_of_Localization_Confidence_for_Accurate_OD/prroi-pooling.jpg?raw=true "prroi pooling")

* Results
  * They train and test on COCO.
  * Their model runs at about 300ms per image on a Titan X.
  * IoU-guided NMS
    * IoU-guided NMS performs slightly better than Soft-NMS and significantly better than classical NMS.
    * IoU-guides NMS shines especially for high IoU APs, improving over Soft-NMS by 1 to 3 percentage points.
  * Optimization-based refinement
    * Using optimization-based refinement also improves APs by around 1 to 3 percentage points (over no refinement).
    * Again, the improvement is most significant for high IoUs, reaching 5 to 6 percentage points better values.
  * Joint Training
    * Training the IoU prediction branch jointly with a base network leads to slightly better features, improving APs by 0.4 to 0.6 points.
    * Combining this with guided-NMS and optimization-based refinement, they improve the AP of a ResNet101+FPN baseline by 2.1 points.
    * ![results joint training](images/Acquisition_of_Localization_Confidence_for_Accurate_OD/results-joint-training.jpg?raw=true "results joint training")

