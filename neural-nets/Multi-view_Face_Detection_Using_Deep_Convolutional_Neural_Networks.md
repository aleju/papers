# Paper

* __Title__: Multi-view Face Detection Using Deep Convolutional Neural Networks
* __Authors__: Sachin Sudhakar Farfade, Mohammad Saberian, Li-Jia Li
* __Link__: [https://arxiv.org/abs/1502.02766](https://arxiv.org/abs/1502.02766)
* __Tags__: Deep Learning, CNN, Face, Detection, DDFD
* __Year__: 2015

# Summary

* What
    * They propose a CNN-based approach to detect faces in a wide range of orientations using a single model. However, since the training set is skewed, the network is more confident about up-right faces.
    * The model does not require additional components such as segmentation, bounding-box regression, segmentation, or SVM classifiers

* How
    * __Data augmentation__: to increase the number of positive samples (24K face annotations), the authors used randomly sampled sub-windows of the images with IOU > 50% and also randomly flipped these images. In total, there were 20K positive and 20M negative training samples.  
    * __CNN Architecture__: 5 convolutional layers followed by 3 fully-connected. The fully-connected layers were converted to convolutional layers. Non-Maximal Suppression is applied to merge predicted bounding boxes.
    * __Training__: the CNN was trained using Caffe Library in the AFLW dataset with the following parameters:
        * Fine-tuning with AlexNet model
        * Input image size = 227x227
        * Batch size = 128 (32+, 96-)
        * Stride = 32
    * __Test__: the model was evaluated on PASCAL FACE, AFW, and FDDB dataset.
    * __Running time__: since the fully-connected layers were converted to convolutional layers, the input image in running time may be of any size, obtaining a heat map as output. To detect faces of different sizes though, the image is scaled up/down and new heatmaps are obtained. The authors found that rescaling image 3 times per octave gives reasonable good performance.
    ![DDFD heatmap](images/DDFD__heatmap.png?raw=true "DDFD heatmap")
    * The authors realized that the model is more confident about up-right faces than rotated/occluded ones. This trend is because the lack of good training examples to represent such faces in the training process. Better results can be achieved by using better sampling strategies and more sophisticated data augmentation techniques.
    ![DDFD example](images/DDFD__example.png?raw=true "DDFD example")
    * The authors tested different strategies for NMS and the effect of bounding-box regression for improving face detection. They NMS-avg had better performance compared to NMS-max in terms of average precision. On the other hand, adding a bounding-box regressor degraded the performance for both NMS strategies due to the mismatch between annotations of the training set and the test set. This mismatch is mostly for side-view faces.

* Results:
    * In comparison to R-CNN, the proposed face detector had significantly better performance independent of the NMS strategy. The authors believe the inferior performance of R-CNN due to the loss of recall since selective search may miss some of the face regions; and loss in localization since bounding-box regression is not perfect and may not be able to fully align the segmentation bounding-boxes, provided by selective search, with the ground truth.
    * In comparison to other state-of-art methods like structural model, TSM and cascade-based methods the DDFD achieve similar or better results. However, this comparison is not completely fair since the most of methods use extra information of pose annotation or information about facial landmarks during the training.