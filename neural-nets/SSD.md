# Paper

* **Title**: SSD: Single Shot MultiBox Detector
* **Authors**: Wei Liu, Dragomir Anguelov, Dumitru Erhan, Christian Szegedy, Scott Reed, Cheng-Yang Fu, Alexander C. Berg
* **Link**: https://arxiv.org/abs/1512.02325
* **Tags**: Neural Network, RCNN
* **Year**: 2015

# Summary

* What
  * They suggest a new bounding box detector.
  * Their detector works without an RPN and RoI-Pooling, making it very fast (almost 60fps).
  * Their detector works at multiple scales, making it better at detecting small and large objects.
  * They achieve scores similar to Faster R-CNN.

* How
  * Architecture
    * Similar to Faster R-CNN, they use a base network (modified version of VGG16) to transform images to feature maps.
    * They do not use an RPN.
    * They predict via convolutions for each location in the feature maps:
      * (a) one confidence value per class (high confidence indicates that there is a bounding box of that class at the given location)
      * (b) x/y offsets that indicate where exactly the center of the bounding box is (e.g. a bit to the left or top of the feature map cell's center)
      * (c) height/width values that reflect the (logarithm of) the height/width of the bounding box
    * Similar to Faster R-CNN, they also use the concept of anchor boxes.
      So they generate the described values not only once per location, but several times for several anchor boxes (they use six anchor boxes).
      Each anchor box has different height/width and optionally scale.
    * Visualization of the predictions and anchor boxes:
      * ![predictions](images/SSD/predictions.jpg?raw=true "predictions")
    * They generate these predictions not only for the final feature map, but also for various feature maps in between (e.g. before pooling layers).
      This makes it easier for the network to detect small (as well as large) bounding boxes (multi-scale detection).
    * Visualization of the multi-scale architecture:
      * ![architecture](images/SSD/architecture.jpg?raw=true "architecture")
  * Training
    * Ground truth bounding boxes have to be matched with anchor boxes (at multiple scales) to determine correct outputs.
      To do this, anchor boxes and ground truth bounding boxes are matched if their jaccard overlap is 0.5 or higher.
      Any unmatched ground truth bounding box is matched to the anchor box with highest jaccard overlap.
    * Note that this means that a ground truth bounding box can be assigned to multiple anchor boxes (in Faster R-CNN it is always only one).
    * The loss function is similar to Faster R-CNN, i.e. a mixture of confidence loss (classification) and location loss (regression).
      They use softmax with crossentropy for the confidence loss and smooth L1 loss for the location.
    * Similar to Faster R-CNN, they perform hard negative mining.
      Instead of training every anchor box at every scale they only train the ones with the highest loss (per example image).
      While doing that, they also pick the anchor boxes to be trained so that 3 in 4 boxes are negative examples (and 1 in 4 positive).
    * Data Augmentation: They sample patches from images using a wide range of possible sizes and aspect ratios.
      They also horizontally flip images, perform cropping and padding and perform some photo-metric distortions.
  * Non-Maximum-Suppression (NMS)
    * Upon inference, they remove all bounding boxes that have a confidence below 0.01.
    * They then apply NMS, removing bounding boxes if there is already a similar one (measured by jaccard overlap of 0.45 or more).

* Results
  * Pascal VOC 2007
    * They achieve around 1-3 points mAP better results than Faster R-CNN.
    * ![results pascal](images/SSD/results_pascal.jpg?raw=true "results pascal")
    * Despite the multi-scale method, the model's performance is still significantly worse for small objects than for large ones.
    * Adding data augmentation significantly improved the results compared to no data augmentation (around 6 points mAP).
    * Using more than one anchor box also had noticeable effects on the results (around 2 mAP or more).
    * Using multiple feature maps to predict outputs (multi-scale architecture) significantly improves the results (around 10 mAP).
      Though adding very coarse (high-level) feature maps seems to rather hurt than help.
  * Pascal VOC 2012
    * Around 4 mAP better results than Faster R-CNN.
  * COCO
    * Between 1 and 4 mAP better results than Faster R-CNN.
  * Times
    * At a batch size of 1, SSD runs at about 46 fps at input resolution 300x300 (74.3 mAP on Pascal VOC) and 19 fps at input resolution 512x512 (76.8 mAP on Pascal VOC).
    * ![results timings](images/SSD/results_timings.jpg?raw=true "results timings")

