# Paper

* **Title**: Group Normalization
* **Authors**: Yuxin Wu, Kaiming He
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Yuxin_Wu_Group_Normalization_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Yuxin_Wu_Group_Normalization_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Normalization, ECCV 2018
* **Year**: 2018

# Summary

* What
  * They propose Group Normalization (GN), a normalization technique similar to Batch Normalization (BN).
  * It works examplewise and splits the channels into groups before normalizing similar to BN.
    * They argue that this is similar to e.g. HOG or SIFT normalizing over groups.
  * As GN works examplewise it is not negatively affected by small batch sizes or batches with non-iid data.
    * This allows to train at tiny batch sizes, opening up GPU memory for larger models.
  * Their normalization is partially motivated from neuroscience, where some models assume normalization across cell responses.

* How
  * There are multiple existing normalization techniques similar to GN, which normalize an `(N, C, H, W)` (examples, channels, height, width) based
    on z-transformations, i.e. `x'_i = (x_i - mean_i)/std_i`. Each normalization technique works in different ways:
    * _Batch Normalization_: Normalizes each channel along the `(N, H, W)` axes, i.e. computes statistics over all examples in the batch.
      Hence it is dependent on the batch size and each example is influenced by all other examples in the batch.
      * BN is the same as Instance Normalization for batch size 1.
    * _Layer Normalization_: Normalizes along the `(C, H, W)` axes, i.e. computes statistics per example (one mean and one standard deviation value per example).
    * _Instance Normalization_: Normalizes long the `(H, W)` axes, i.e. computes one mean and standard deviation per example and channel.
      Other examples or channels do not influence the computation.
    * _Group Normalization_ (their method): Normalizes roughly along the `(C, H, W)` axes, similar to Layer Normalization.
      However, the channel axis is subdivided into groups of channels and the normalization happens over all channels within the same group
      (instead of over all channels in the example as in Layer Normalization).
      * GN becomes Instance Normalization when the number of groups is set to the number of channels.
      * GN becomes Layer Normalization when the number of groups is set to 1.
    * Visualization of the relationship:
      * ![comparison](images/Group_Normalization/comparison.jpg?raw=true "comparison")

* Results
  * They test on ImageNet, training on 8 GPUs. They use ResNet-50. Their default number of groups in GN is 32.
  * At batch size 32: They observe 0.5 percentage points higher validation error with GN as opposed to BN (layer normalization: 1.7 points worse, instance normalization: 4.8 points worse).
    * They observe slightly lower training error of GN compared to BN.
      They argue that BN profits from a regularizing effect from the examples in each batch being randomly picked (i.e. mean and variance are a bit stochastic).
  * At batch size 2: GN's accuracy remains constant (compared to batch size 32), while BN degrades significantly. GN beats BN by 10.6 points.
    * They also compare to Batch Renormalization with well chosen hyperparameters and find that it improves BN's performance, but it still performs 2.1 points worse than GN.
  * Accuracy vs. batch size:
    * ![sensitivity batch size](images/Group_Normalization/sensitivity_batch_size.jpg?raw=true "sensitivity batch size")
  * Channels per group
    * They found on ImageNet (with ResNet-50) that around 8 or more channels per group performs best.
    * ![group division](images/Group_Normalization/group_division.jpg?raw=true "group division")
  * They get similar results when training with ResNet-101.
  * VGG-16
    * They trained VGG-16 on ImageNet with no normalization, BN and GN.
    * They observed that the feature maps before normalization and ReLU had a significantly wider value range (-80 to +20)
      when using no normalization as opposed to using normalization (BN: -3 to +3, GN: -4 to +3).
  * Object Detection and Segmentation on COCO
    * They compare BN and GN for object detection and segmentation on COCO using Mask R-CNN with different backbones.
    * ResNet-50 backbone
      * Using GN instead of (frozen) BN improves box AP by 1.1 points and mask AP by 0.8 points.
    * Feature Pyramid Network (FPN) backbone
      * They compare using BN and GN only in the head of Mask R-CNN.
        BN performs significantly worse (9 points) than GN, even though the batch size is at 512 RoIs.
        They argue that this is because the examples are not iid (they all come from the same image).
      * GN in the backbone improves AP by 0.5 points.
      * Using GN also allows them to train the whole model from scratch (i.e. no pretraining) without training instabilities.
        They reach scores very close to the pretrained versions.
  * Video classification on Kinetics
    * They test on the Kinetics dataset, which requires classifying videos and hence leads to large memory consumption per example.
    * They use a ResNet-50 Inflated 3D (I3D) network as baseline.
    * Normalization is extended from (H,W) axes to (T,H,W) axes, adding the temporal dimension.
    * GN performs overall superior to BN by up to 1.2 points accuracy.
    * GN profits from *not* sub-sampling frames (e.g. using only every second frame as input).
      BN however does not improve its accuracy from deactivating subsampling, because that also required reducing the batch size from 8 to 4 examples.
      The positive effect of deactivating subsampling was masked by the negative effect of reducing the batch size,
      leading to a false conclusion that subsampling does not degrade accuracy.

