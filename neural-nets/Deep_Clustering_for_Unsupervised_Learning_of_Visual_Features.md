# Paper

* **Title**: Deep Clustering for Unsupervised Learning of Visual Features
* **Authors**: Mathilde Caron, Piotr Bojanowski, Armand Joulin, Matthijs Douze
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Mathilde_Caron_Deep_Clustering_for_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Mathilde_Caron_Deep_Clustering_for_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Unsupervised Learning, Clustering, ECCV 2018
* **Year**: 2018

# Summary

* What
  * They describe a strategy to train CNNs in unsupervised fashion.
  * The method is based on iterating k-means clustering in feature space, followed by training the CNN to predict the cluster labels.
  * The method achieves sizeable improvements over the state of the art in unsupervised learning on ImageNet.

* How
  * Method
    * They start with a CNN (not pretrained)
    * They then iterate the following steps:
      1. They apply the CNN to their dataset, converting each image to a feature vector. (At the start, this will be mostly noise.)
      2. They apply k-means to these feature vectors.
      3. They use the resulting clustering as (pseudo-)labels for the images.
      4. They train the CNN for some batches to predict these (pseudo-)labels.
      5. They start again at (1).
    * Visualization:
      * ![method](images/Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features/method.jpg?raw=true "method")
  * Avoiding trivial solutions
    * Empty Clusters
      * While k-means is iterating, some clusters may become empty.
      * If that happens, they move the centroid of such a cluster (e.g. call it "cluster A" here) to another cluster (e.g. "cluster B"), where B must have associated feature vectors.
      * After moving, they perturb the centroid of cluster A's location by a small amount.
      * They then split the feature vectors of cluster B between A and B.
    * Trivial Parameterization
      * If one cluster becomes dominating, i.e. if most of the feature vectors are associated to it,
        the CNN can start to learn trivial parameterizations that only focus on differentiating between that one cluster and everything else,
        because.
      * They avoid this by sampling images based on a uniform distribution over the (pseudo-)labels.
  * Other stuff
    * Their test their methods mainly with AlexNet. Also a bit with VGG. (Both modified to use BN.)
    * They update the clusters once per epoch on ImageNet.
    * They train on ImageNet for 500 epochs.
    * They apply a Sobel-based filter before their models (i.e. they do not get colors as inputs).

* Results
  * ImageNet
    * They measure the normalized mutual information between cluster assignments and ImageNet labels.
      This mutual information increases over time, indicating that the cluster assignments matches more and more the labels (which were not used during training).
    * They measure the normalized mutual information of cluster assignments between each pair of two consecutive epochs.
      This mutual information increases over time, indicating that the clustering becomes more stable.
    * They measure the achieved mAP on Pascal VOC 2007 (I guess they use the model as pretrained weights?)
      based on the number of clusters in k-means.
      They achieve best results with 10,000 clusters.
    * Accuracy per layer
      * After training they place a linear classifier on top of their model and train it on the labels. They do repeat this for each layer to get the accuracy per layer.
      * They significantly outperform all competing methods (by several percentage points, depending on layer).
      * At conv5 (max conv layer in AlexNet) they are beaten by supervised learning by ~12 points (50.5 vs 38.2).
      * The difference is lower for conv3 (44.2 vs 41.0).
      * They also compare models pretrained on ImageNet, with the linear classifier fine-tuned on the Places dataset.
        In this case they can keep up more with the model pretrained in supervised fashion (38.7 vs 34.7 at conv5, 39.4 vs 39.8 at conv4).
    * Stats:
      * ![results imagenet](images/Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features/results_imagenet.jpg?raw=true "results imagenet")
  * YFCC100M
    * They train on ImageNet and then search on YFCC100M for images leading to high activations in filters.
    * Images with high activations and generated "ideal" images for that filter:
      * ![YFCC100M filters](images/Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features/yfcc100m_filters.jpg?raw=true "YFCC100M filters")
    * Images with high activations for filters in layer conv5:
      * ![YFCC100M filters conv5](images/Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features/yfcc100m_filters_conv5.jpg?raw=true "YFCC100M filters conv5")
      * They note that some filters in conv5 seem to just re-learn patterns that are already covered by filters in lower layers.
  * Pascal VOC
    * They pretrain on ImageNet and then finetune on Pascal VOC for classification, object detection and segmentation.
    * They are still behind a model pretrained with labels. Depending on the task between ~1 (detection), ~3 (segmentation) or ~6 (classification) points.
    * They outperform other unsupervised methods by several points.
    * They outperform a randomly initialized (i.e. not pretrained) model significantly.
      (This is even more the case if only the last fully connected layers are finetuned, not the convolutional layers.)
    * They train with VGG16 instead of AlexNet and can improve their accuracy by an amount comparable to when the same model switch is done for the supervised training.

