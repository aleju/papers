# Paper

* **Title**: Arguing Machines: Perception-Control System Redundancy and Edge Case Discovery in Real-World Autonomous Driving
* **Authors**: Lex Fridman, Benedikt Jenik, Bryan Reimer
* **Link**: https://arxiv.org/abs/1710.04459
* **Tags**: Neural Network, self-driving
* **Year**: 2017

# Summary

* What
  * They present a method to detect hard edge cases in datasets of self-driving cars.
  * These are cases that might lead to accidents.
  * The method is based on running two models simultaneously and measuring their disagreement with each other.

* How
  * They use two different models:
    * Tesla-AD: Proprietary autopilot from Tesla.
    * CNN: Their own CNN, trained on a Tesla dataset (images from car's perspective + steering & acceleration annotation; 420 hours of driving).
      The network's architecture is similar to the one in the NVIDIA paper.
  * Their considered five different inputs for their own CNN:
    * **M5**: RGB images (same as in the NVIDIA paper)
    * **M4**: Images of edges in each color channel (i.e. three edge maps).
      (No detailed explanation on how the edges were detected.)
    * **M3**: Grayscale images, from `t-20`, `t-10` and `t` (where `t` is the current frame).
    * **M2**: Grayscale difference images `t - t-10` (i.e. difference between the current frame and the one 10 frames ago), `t - t-5`, `t - t-1`.
    * **M1**: Grayscale difference images `t-20 - t-30`, `t-10 - t-20`, `t - t-10`.
    * Visualization:
      * ![inputs](images/Arguing_Machines/inputs.jpg?raw=true "inputs")
  * Training
    * They split the data into subgroups, each being defined up by the steering wheel angle.
      E.g. all example with an angle in the range `[-10, 10]` end in one group.
    * They select equally from all groups and get 100k training and 50k validation images.
    * This prevents the model from only training on examples showing straight driving.
  * Disagreement
    * Their intention is to find hard example / edge cases.
    * They do this by measuring the disagreement between the models (*Tesla-AD* and *CNN*).
    * To do that, they first clip the predicted angles to the range `[-10, 10]`.
      Then they normalize the result to the range `[-1, 1]`.
    * They sum the differences of these predictions over a one second window (30 examples).
    * If the sum exceeds a threshold `delta`, they consider the models to be in disagreement and the example to be an edge case.
    * They use `delta=10`.

* Results
  * Input M1 performed best, followed by M2, M3, M4 and M5 (in that order).
    (Measured via MAE of CNN predictions vs. ground truth annotations.)
  * They can predict with 90% accuracy whether there will be a disengagement in the next 5 seconds (i.e. human driver took control in the dataset).
