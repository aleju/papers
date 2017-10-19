# Paper

* **Title**: Image Crowd Counting Using Convolutional Neural Network and Markov Random Field
* **Authors**: Kang Han, Wanggen Wan, Haiyan Yao, Li Hou
* **Link**: https://arxiv.org/abs/1706.03686
* **Tags**: Neural Network, ResNet, MRF
* **Year**: 2017

# Summary

* What
  * They suggest a method to estimate the number of visible people in images of crowds.
  * Their method is based on a combination of CNNs and MRFs.

* How
  * They split each image of crowds into overlapping square patches.
  * Each patch is resized to 224x224 and fed through ResNet-152. (Sounds like they don't fine-tune that model.)
  * They extract the features from the last layer `fc1000`. (A bit weird considering those are the class probabilities.)
  * They apply a few fully connected layers to that result in order to arrive at one value, which regresses the number of people visible in that patch.
  * Those last fully connected layers are trained via a simple mean squared (or also absolute) error.
    The ground truth labels are computed from the total count of visible people in the image (sum of patch counts equals total number of people).
  * They argue that count values should be similar between adjacent patches and smoothen them using a MRF:
    * ![mrf](images/Image_Crowd_Counting_using_CNN_and_MRF/mrf.jpg?raw=true "mrf")
    * `p` is a patch (same for `q`) and `P` are all patches
    * `c_p`/`c_q` is the count of patch p/q
    * `D_p(c_p)` is the cost of assigning count `c_p` to patch `p`, usually given by `lambda * (ground_truth - c_p)^2`, where `lambda` is a trained(?) weight (per patch?)
    * `V(c_p - c_q)` is the smoothness cost, usually given by `(c_p - c_q)^2`
  * Training goal of the MRF formulation is to minimize the energy `E(c)`,
    which is achieved by mostly assigning ground truth counts (`D(.)`), but also keeping the counts smooth to the neighbors (`V(.)`).
  * The energy function is minimized using belief propagation.
    (But doesn't that have to be reexecuted for every single image?
     Only `lambda` is apparently trained, which is image-specific.
     That would probably be slow and requires the usage of ground truth annotation for all images, including validation/test?!)
  * Visualization of the steps:
    * ![steps](images/Image_Crowd_Counting_using_CNN_and_MRF/steps.jpg?raw=true "steps")

* Results
  * UCF dataset
    * Contains 50 grayscale images with 63705 annotated people (each head is annotated).
    * They achieve the best score among all competitors (about 10% lower in MAE compared to best alternative).
  * Shanghaitech dataset
    * Contains 1198 images with 330165 annotated people (also head annotations).
    * They achieve the best score among all competitors (about 30% lower in MAE compared to best alternative).
  * Visualization of smoothing from MRF:
    * ![mrf smoothing](images/Image_Crowd_Counting_using_CNN_and_MRF/mrf_smoothing.jpg?raw=true "mrf smoothing")

