# Paper

* **Title**: HyperFace: A Deep Multi-task Learning Framework for Face Detection, Landmark Localization, Pose Estimation, and Gender Recognition
* **Authors**: Rajeev Ranjan, Vishal M. Patel, Rama Chellappa
* **Link**: http://arxiv.org/abs/1603.01249v2
* **Tags**: Neural Network, face
* **Year**: 2016

# Summary

* What
  * They suggest a single architecture that tries to solve the following tasks:
    * Face localization ("Where are faces in the image?")
    * Face landmark localization ("For a given face, where are its landmarks, e.g. eyes, nose and mouth?")
    * Face landmark visibility estimation ("For a given face, which of its landmarks are actually visible and which of them are occluded by other objects/people?")
    * Face roll, pitch and yaw estimation ("For a given face, what is its rotation on the x/y/z-axis?")
    * Face gender estimation ("For a given face, which gender does the person have?")

* How
  * *Pretraining the base model*
    * They start with a basic model following the architecture of AlexNet.
    * They train that model to classify whether the input images are faces or not faces.
    * They then remove the fully connected layers, leaving only the convolutional layers.
  * *Locating bounding boxes of face candidates*
    * They then use a [selective search and segmentation algorithm](https://www.robots.ox.ac.uk/~vgg/rg/papers/sande_iccv11.pdf) on images to extract bounding boxes of objects.
    * Each bounding box is considered a possible face.
    * Each bounding box is rescaled to 227x227.
  * *Feature extraction per face candidate*
    * They feed each bounding box through the above mentioned pretrained network.
    * They extract the activations of the network from the layers `max1` (27x27x96), `conv3` (13x13x384) and `pool5` (6x6x256).
    * They apply to the first two extracted tensors (from max1, conv3) convolutions so that their tensor shapes are reduced to 6x6xC.
    * They concatenate the three tensors to a 6x6x768 tensor.
    * They apply a 1x1 convolution to that tensor to reduce it to 6x6x192.
    * They feed the result through a fully connected layer resulting in 3072-dimensional vectors (per face candidate).
  * *Classification and regression*
    * They feed each 3072-dimensional vector through 5 separate networks:
      1. Detection: Does the bounding box contain a face or no face. (2 outputs, i.e. yes/no)
      2. Landmark Localization: What are the coordinates of landmark features (e.g. mouth, nose, ...). (21 landmarks, each 2 values for x/y = 42 outputs total)
      3. Landmark Visibility: Which landmarks are visible. (21 yes/no outputs)
      4. Pose estimation: Roll, pitch, yaw of the face. (3 outputs)
      5. Gender estimation: Male/female face. (2 outputs)
    * Each of these network contains a single fully connected layer with 512 nodes, followed by the output layer with the above mentioned number of nodes.
  * *Architecture Visualization*:
    * ![Architecture](images/HyperFace__architecture.jpg?raw=true "Architecture")
  * *Training*
    * The base model is trained once (see above).
    * The feature extraction layers and the five classification/regression networks are trained afterwards (jointly).
    * The loss functions for the five networks are:
      1. Detection: BCE (binary cross-entropy). Detected bounding boxes that have an overlap `>=0.5` with an annotated face are considered positive samples, bounding boxes with overlap `<0.35` are considered negative samples, everything in between is ignored.
      2. Landmark localization: Roughly MSE (mean squared error), with some weighting for visibility. Only bounding boxes with overlap `>0.35` are considered. Coordinates are normalized with respect to the bounding boxes center, width and height.
      3. Landmark visibility: MSE (predicted visibility factor vs. expected visibility factor). Only for bounding boxes with overlap `>0.35`.
      4. Pose estimation: MSE.
      5. Gender estimation: BCE.
  * *Testing*
    * They use two postprocessing methods for detected faces:
      * Iterative Region Proposals:
        * They localize landmarks per face region.
        * Then they compute a more appropriate face bounding box based on the localized landmarks.
        * They feed that new bounding box through the network.
        * They compute the face score (face / not face, i.e. number between 0 and 1) for both bounding boxes and choose the one with the higher score.
        * This shrinks down bounding boxes that turned out to be too big.
        * The method visualized:
          * ![IRP](images/HyperFace__irp.jpg?raw=true "IRP")
      * Landmarks-based Non-Maximum Suppression:
        * When multiple detected face bounding boxes overlap, one has to choose which of them to keep.
        * A method to do that is to only keep the bounding box with the highest face-score.
        * They instead use a median-of-k method.
        * Their steps are:
          1. Reduce every box in size so that it is a bounding box around the localized landmarks.
          2. For every box, find all bounding boxes with a certain amount of overlap.
          3. Among these bounding boxes, select the `k` ones with highest face score.
          4. Based on these boxes, create a new box which's size is derived from the median coordinates of the landmarks.
          5. Compute the median values for landmark coordinates, landmark visibility, gender, pose and use it as the respective values for the new box.

* Results
  * Example results:
    * ![Example results](images/HyperFace__example_results.jpg?raw=true "Example results")
  * They test on AFW, AFWL, PASCAL, FDDB, CelebA.
  * They achieve the best mean average precision values on PASCAL and AFW (compared to selected competitors).
  * AFW results visualized:
    * ![AFW](images/HyperFace__afw.jpg?raw=true "AFW")
  * Their approach achieve good performance on FDDB. It has some problems with small and/or blurry faces.
  * If the feature fusion is removed from their approach (i.e. extracting features only from one fully connected layer at the end of the base network instead of merging feature maps from different convolutional layers), the accuracy of the predictions goes down.
  * Their architecture ends in 5 shallow networks and shares many layers before them. If instead these networks share no or few layers, the accuracy of the predictions goes down.
  * The postprocessing of bounding boxes (via Iterative Region Proposals and Landmarks-based Non-Maximum Suppression) has a quite significant influence on the performance.
  * Processing time per image is 3s, of which 2s is the selective search algorithm (for the bounding boxes).
