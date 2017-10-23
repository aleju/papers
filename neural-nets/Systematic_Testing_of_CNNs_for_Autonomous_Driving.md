# Paper

* **Title**: Systematic Testing of Convolutional Neural Networks for Autonomous Driving
* **Authors**: Tommaso Dreossi, Shromona Ghosh, Alberto Sangiovanni-Vincentelli, Sanjit A. Seshia
* **Link**: https://arxiv.org/abs/1708.03309
* **Tags**: Neural Network, self-driving
* **Year**: 2017

# Summary

* What
  * They suggest a framework that can be used to test CNNs (for self-driving cars).
  * The framework makes use of synthetically generated examples.

* How
  * Their framework is split into three parts: Image generator, sampling methods and visualization tools.
  * Image generator
    * The image generator uses real images from a dataset as input.
    * It may change these images via basic transformations (e.g. brightness, saturation) or by drawing objects on them (e.g. cars).
    * Though they seem to only apply contrast changes and project cars (at x/y-positions).
      The car projection also makes use of the road location in order to draw cars smaller if they are further away.
    * Visualization:
      * ![image generation](images/Systematic_Testing_of_CNNs_for_Autonomous_Driving/image_generation.jpg?raw=true "image generation")
  * Sampling methods
    * Each modification applied by the image generator can be expressed as one or more components of a vector (e.g. 2.0 for "increase brightness to 2x the initial value").
    * Such a generated example/vector is then a point in a space. The space contains all possible images that can be generated.
    * Generating images and testing a CNN on them is expensive. So ideally one wants have a good search method to find images that confuse the CNN.
    * They generate candidate points in the example space using Halton and lattice-based sequences.
      (Uniform sampling of candidate points would be possible, but would not cover the space optimally.)
    * Using that method, they can generate example inputs and the corresponding CNN scores.
    * They then train a gaussian process model on these `(image, CNN score)` pairs.
    * This allows them to more efficiently generate images that probably cause the CNN to fail.
  * Visualization tools
    * They generate plots from example/input vectors and measured scores (confidence, IoU).

* Results
  * Example results for SqueezeDet and Yolo bounding box detectors, applied to a single test image with a car projected onto various locations:
    * ![results street](images/Systematic_Testing_of_CNNs_for_Autonomous_Driving/results_street.jpg?raw=true "results street")
    * The size of each point indicates the IoU, i.e. small points indicate that the CNN missed the car or predicted a bad bounding box).
    * Blue points indicate that the network predicted a low confidence value.
    * SqueezeDet seems to have problems with cars at the center and right of the roud.
    * Yolo's confidence value seem to follow the distance of the car. Its achieved IoUs seem to be lower in general with a blind spot on the right of the road.
  * Corresponding 3d plots of points, organized by x/y position of the car, IoU and confidence value (:
    * ![results 3d plots](images/Systematic_Testing_of_CNNs_for_Autonomous_Driving/results_3dplots.jpg?raw=true "results 3d plots")

