# Paper

* **Title**: You Only Look Once: Unified, Real-Time Object Detection
* **Authors**: Joseph Redmon, Santosh Divvala, Ross Girshick, Ali Farhadi
* **Link**: https://arxiv.org/abs/1506.02640
* **Tags**: Neural Network, RCNN
* **Year**: 2015

# Summary

* What
  * They suggest a model ("YOLO") to detect bounding boxes in images.
  * In comparison to Faster R-CNN, this model is faster but less accurate.
* How
  * Architecture
    * Input are images with a resolution of 448x448.
    * Output are `S*S*(B*5 + C)` values (per image).
      * `S` is the grid size (default value: 7). Each image is split up into `S*S` cells.
      * `B` is the number of "tested" bounding box shapes at each cell (default value: 2).
        So at each cell, the network might try one large and one small bounding box.
        The network predicts additionally for each such tested bounding box `5` values.
        These cover the exact position (x, y) and scale (height, width) of the bounding box as well as a confidence value.
        They allow the network to fine tune the bounding box shape and reject it, e.g. if there is no object in the grid cell.
        The confidence value is zero if there is no object in the grid cell and otherwise matches the IoU between predicted and true bounding box.
      * `C` is the number of classes in the dataset (e.g. 20 in Pascal VOC). For each grid cell, the model decides once to which of the `C` objects the cell belongs.
    * Rough overview of their outputs:
      * ![Method](images/YOLO__method.jpg?raw=true "Method")
    * In contrast to Faster R-CNN, their model does *not* use a separate region proposal network (RPN).
    * Per bounding box they actually predict the *square root* of height and width instead of the raw values.
      That is supposed to result in similar errors/losses for small and big bounding boxes.
    * They use a total of 24 convolutional layers and 2 fully connected layers.
      * Some of these convolutional layers are 1x1-convs that halve the number of channels (followed by 3x3s that double them again).
      * Overview of the architecture:
        * ![Architecture](images/YOLO__architecture.jpg?raw=true "Architecture")
    * They use Leaky ReLUs (alpha=0.1) throughout the network. The last layer uses linear activations (apparently even for the class prediction...!?).
    * Similarly to Faster R-CNN, they use a non maximum suppression that drops predicted bounding boxes if they are too similar to other predictions.
  * Training
    * They pretrain their network on ImageNet, then finetune on Pascal VOC.
    * Loss
      * They use sum-squared losses (apparently even for the classification, i.e. the `C` values).
      * They dont propagate classification loss (for `C`) for grid cells that don't contain an object.
      * For each grid grid cell they "test" `B` example shapes of bounding boxes (see above).
        Among these `B` shapes, they only propagate the bounding box losses (regarding x, y, width, height, confidence) for the shape that has highest IoU with a ground truth bounding box.
      * Most grid cells don't contain a bounding box. Their confidence values will all be zero, potentialle dominating the total loss.
        To prevent that, the weighting of the confidence values in the loss function is reduced relative to the regression components (x, y, height, width).
* Results
  * The coarse grid and B=2 setting lead to some problems. Namely, small objects are missed and bounding boxes can end up being dropped if they are too close to other bounding boxes.
  * The model also has problems with unusual bounding box shapes.
  * Overall their accuracy is about 10 percentage points lower than Faster R-CNN with VGG16 (63.4% vs 73.2%, measured in mAP on Pascal VOC 2007).
  * They achieve 45fps (22ms/image), compared to 7fps (142ms/image) with Faster R-CNN + VGG16.
  * Overview of results on Pascal VOC 2012:
    * ![Results on VOC2012](images/YOLO__results.jpg?raw=true "Results on VOC2012")
  * They also suggest a faster variation of their model which reached 145fps (7ms/image) at a further drop of 10 percentage points mAP (to 52.7%).
  * A significant part of their error seems to come from badly placed or sized bounding boxes (e.g. too wide or too much to the right).
  * They mistake background less often for objects than Fast R-CNN. They test combining both models with each other and can improve Fast R-CNN's accuracy by about 2.5 percentage points mAP.
  * They test their model on paintings/artwork (Picasso and People-Art datasets) and notice that it generalizes fairly well to that domain.
  * Example results (notice the paintings at the top):
    * ![Examples](images/YOLO__examples.jpg?raw=true "Examples")
