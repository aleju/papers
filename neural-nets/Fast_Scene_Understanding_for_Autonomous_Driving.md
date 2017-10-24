# Paper

* **Title**: Fast Scene Understanding for Autonomous Driving
* **Authors**: Davy Neven, Bert De Brabandere, Stamatios Georgoulis, Marc Proesmans, Luc Van Gool
* **Link**: https://arxiv.org/abs/1708.02550
* **Tags**: Neural Network, self-driving, segmentation
* **Year**: 2017

# Summary

* What
  * They suggest a method to predict for images the semantic segmentation maps, instance segmentation maps and depth maps.
    * Semantic segmentation = "assign color X to all pixels showing a car, color Y to all pixels showing people, ..."
    * Instance segmentation = "assign color X to all pixels showing car number 1, color Y to all pixels showing car number 2, ..."
  * Their method is optimized to run fast and with low memory demands.
  * The method is aimed at self-driving cars.

* How
  * Architecture
    * They base their model on ENet.
    * ENet is a network for fast segmentation.
      It uses three blocks of (several) residual convolutions for downscaling, followed by two blocks of upscaling.
      Some convolutions are dilated.
      Number of filters does not exceed 128.
    * They share two blocks of downscaling between all three branches (semantic segmentation / instance segmentation / depth maps).
    * Each branch gets one unique downscaling module and two upscaling modules.
  * Losses
    * Semantic Segmentation: Pixelwise cross-entropy.
    * Instance Segmentation:
      * They do not directly predict some kind of instance flag per pixel.
      * Instead they generate per pixel an embedding vector.
        These are supposed to be similar for pixels belonging to the same instance (and dissimilar for different instances).
        They can then cluster these vectors to find all pixels belonging to an instance (they use GPU based mean shift clustering for that).
      * In order to train the network to generate such embeddings,
        they view each instance as a class and design the losses so that the intra-class variance (distances) is minimized,
        while the intra-class distances are maximized.
      * So they get a variance term/loss and a distance term/loss.
        Both of these are hinged (so e.g. once the variance goes below a certain threshold, the corresponding loss just becomes zero).
      * ![instance seg loss](images/Fast_Scene_Understanding_for_Autonomous_Driving/instance_seg_loss.jpg?raw=true "instance seg loss")
      * `L_var = intraclass variance loss`, `L_dist = interclass variance loss`, `L_reg = regularization loss`
      * `||.|| = L2 distance`, `[.]_+ = max(0, x) = hinge`
      * `C = class`, `N_c = pixels in class`, `mu_c = mean embedding`, `x_i = pixel embedding`, `delta_v = variance hinge`, `delta_d = interclass distance hinge`
    * Depth Estimation:
      * Common loss here (according to the authors) would be the L2 distance with a scale invariance term and local structure similarity term.
        But they don't use these as it performs worse than their loss.
      * Instead they use a reverse Huber loss, which seems to have some similarity with a combination of L1 and L2 loss:
        * ![depth loss](images/Fast_Scene_Understanding_for_Autonomous_Driving/depth_loss.jpg?raw=true "depth loss")
  * Other
    * Input image size is 1024x512.
    * They use Adam with learning rate 5e-4 and batch size 10.
    * They keep the batch norm parameters fixed (no explanation why).

* Results
  * For semantic segmentation on Cityscapes they reach the same score as ENet (59.3 class IoU, 80.4 category IoU).
  * For instance segmentation on Cityscapes they reach 21.0 AP as opposed to 46.9 AP of Mask R-CNN.
  * For depth map prediction on Cityscapes their result differ by distance of objects.
    The difference is lower for close objects (MAE of 1.5m for objects `<25m` away) and higher for the ones further apart (MAE of 7.5m for objects `<100m` away).
  * Visualization of predicted depth vs real depth:
    * ![depth prediction results](images/Fast_Scene_Understanding_for_Autonomous_Driving/depth_prediction_results.jpg?raw=true "depth prediction results")
  * Example results:
    * ![example results](images/Fast_Scene_Understanding_for_Autonomous_Driving/example_results.jpg?raw=true "example results")
  * They reach 21fps (that should be around 3x or so faster than Mask R-CNN) and 1.2GB memory footprint.
  * Training all branches jointly in one network (as described above) vs. training them completely separately (fully disjoint networks) improves accuracy by a bit.


