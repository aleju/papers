# Paper

* **Title**: Deep Continuous Fusion for Multi-Sensor 3D Object Detection
* **Authors**: Ming Liang, Bin Yang, Shenlong Wang, Raquel Urtasun
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Ming_Liang_Deep_Continuous_Fusion_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Ming_Liang_Deep_Continuous_Fusion_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Object Detection, 3D, 2D, Sensor Fusion, Point Cloud, LiDAR, Self-Driving Cars, ECCV 2018
* **Year**: 2018

# Summary

* What
  * They propose an object detector that fuses 2D information (from images) and 3D information (from LiDAR, i.e. point cloud) to predict 3D objects in birds eye view (BEV).
  * Their method is based on projecting 2D images into point clouds (usually it's the other way round, i.e. point clouds are projected into 2D images).
  * They propose a layer to perform the 2D-3D fusion at multiple image scales.
  * The result is a fast and fairly accurate object detector.

* How
  * Basic Architecture
    * They feed the images through a ResNet18 branch (initialized via ImageNet pretraining).
    * They feed the point cloud through a custom residual network.
      * Input is the voxelized point cloud in BEV.
      * The network is comparable to ResNet18.
      * They use a total of 36 convolutions in five blocks, with a growing number of convolutions per block and downscaling at the start of each block.
      * This network is initialized via Xavier initialization.
    * They features extracted by the image branch are fused at multiple scales into the point cloud BEV branch using Continuous Fusion layers.
    * The output of the point cloud BEV branch is comparable to many other object detectors:
      * Per spatial location classification (object class + background) and regression predictions (x, y, z and width, length, height).
      * They use two anchor boxes, one for objects with a rotation of 0 degrees and one for objects with a rotation of 90 degrees.
      * As usually, they apply Non-Maximum-Suppression to the resulting bounding boxes.
    * Visualization:
      * ![architecture](images/Deep_Continuous_Fusion_for_Multi-Sensor_3D_Object_Detection/architecture.jpg?raw=true "architecture")
  * Continuous Fusion Layer
    * The Continuous Fusion Layer fuses 2D (camera) and 3D (LiDAR) features.
    * Its basic principle is to project 2D information into 3D. (Counterintuitively, it still performs first a projection from 3D to 2D.)
    * As their network works in BEV, the fusion merges information from a dense 2D grid (camera) with another dense 2D grid (LiDAR in BEV) instead of a point cloud.
    * Assume you have a LIDAR BEV feature map with one channel, i.e. a 2D grid.
      You want to merge information from the camera into one single spatial location (e.g. column 10, row 20) of the grid.
      The following steps are then performed:
      1. For the given spatial location, find the K nearest neighbour points in the BEV point cloud. (E.g. find the K neighbours of y=10, x=20.)
      2. Re-add the z coordinate to each of these K nearest neighbours (projects them from BEV 3D back to full 3D).
      3. Project these 3D points onto the 2D image plane. This indicates which image areas might contain information relevant for the 3D BEV location.
      4. Use bilinear interpolation to estimate the image's features at the projected locations.
         For each location also estimate the offset vector from the BEV spatial location (e.g. y=10, x=20) to the 3D point (I guess in BEV?).
      5. Concat the feature vector and offset vector from step (4) to one vector.
         Then apply fully connected layers to that (in their case three layers).
         This generates a vector per projected point.
         Sum these vectors.
         Concat the resulting vector to the point cloud BEV feature map from step (1) at the chosen starting location (e.g. y=10, x=20).
    * Visualization of the steps:
      * ![continuous fusion](images/Deep_Continuous_Fusion_for_Multi-Sensor_3D_Object_Detection/continuous_fusion.jpg?raw=true "continuous fusion")
  * Other stuff
    * As is common, their classification loss is binary cross entropy and their regression loss is smooth L1.
    * As is common, they encode the x-, y- and z-coordinates (predicted via the regression branch) is relative offsets to the anchor's center.
    * As is common, they encode the widths, lengths and heights logarithmically, e.g. `log(width/width_anchor)`.
    * They use Adam without weight decay.
    * For training on KITTI, they augment the dataset, e.g. via scaling, translation and rotation of the point cloud and camera image
      (matched to each other, so that projections from one sensor to the other remain sensible).
    * They crop the point cloud to 70m in front of the ego vehicle and 40m to the left and right.
    * They voxelize the point cloud to 512x448x32 voxels. (Sounds like they apply 2D convs to an input with 32 channels encoding height information.)

* Results
  * KITTI
    * Note: They only evaluate on the class "car", arguing that -- in the case of their model -- there is not enough training data for the other classes in KITTI.
    * They achieve accuracies roughly comparable with other well-performing 3D object detectors. (Though not state of the art.)
    * Their detector runs at about 15fps or 66ms per scene, beating most other models.
      * They are roughly twice as fast as AVOD-FPN, but quite a bit less accurate (~5 percentage points for moderate 3D AP).
    * Using only LiDAR (and not images) as input significantly worsens results.
    * Using a fusion strategy similar in which only the BEV locations closest to 3D points are projected to the image plane (no K-NN) and without using offset vectors significantly worsens results.
    * Removing the offset vector significantly worsens results.
    * Picking K>1 for the K-nearest-neighbours search does not produce better results than K=1.
    * Limiting the maximum distance during the K-nearest-neighbour search to 10m has only a tiny positive impact on AP.
    * Ablation results ("geometric feature" is the offset vector):
      * ![KITTI ablation results](images/Deep_Continuous_Fusion_for_Multi-Sensor_3D_Object_Detection/kitti_ablation_results.jpg?raw=true "KITTI ablation results")
 * TOR4D (Uber dataset)
    * They evaluate here on more classes and increase the forward distance to 100m.
    * They decrease the parameters in the network to keep the time per scene roughly comparable (despite increase in max forward distance).
    * They evaluate the achieved AP by class and distance:
      * ![TOR4D AP by distance](images/Deep_Continuous_Fusion_for_Multi-Sensor_3D_Object_Detection/tor4d_ap_by_distance.jpg?raw=true "TOR4D AP by distance")

