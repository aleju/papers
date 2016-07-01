# Paper

* **Title**: Convolutional Pose Machines
* **Authors**: Shih-En Wei, Varun Ramakrishna, Takeo Kanade, Yaser Sheikh
* **Link**: https://arxiv.org/abs/1602.00134
* **Tags**: Neural Network, pose
* **Year**: 2016

# Summary

* What
  * They suggest a new model for human pose estimation (i.e. to lay a "skeleton" over the image of a person).
  * Their model has a (more or less) recurrent architecture.
    * Initial estimates of keypoint locations are refined in several steps.
    * The idea of the recurrent architecture is derived from message passing, unrolled into one feed-forward model.

* How
  * Architecture
    * They generate the end result in multiple steps, similar to a recurrent network.
    * Step 1:
      * Receives the image (368x368 resolution).
      * Applies a few convolutions to the image in order to predict for each pixel the likelihood of belonging to a keypoint (head, neck, right elbow, ...).
    * Step 2 and later:
      * (Modified) Receives the image (368x368 resolution) and the previous likelihood scores.
      * (Same) Applies a few convolutions to the image in order to predict for each pixel the likelihood of belonging to a keypoint (head, neck, right elbow, ...).
      * (New) Concatenates the likelihoods with the likelihoods of the previous step.
      * (New) Applies a few more convolutions to the concatenation to compute the final likelihood scores.
    * Visualization of the architecture:
      * ![Architecture](images/Convolutional_Pose_Machines__architecture.jpg?raw=true "Architecture")
  * Loss function
    * The basic loss function is a simple mean squared error between the expected output maps per keypoint and the predicted ones.
    * In the expected output maps they mark the correct positions of the keypoints using a small gaussian function.
    * They apply losses after each step in the architecture, argueing that this helps against vanishing gradients (they don't seem to be using BN).
    * The expected output maps of the first step actually have the positions of all keypoints of a certain type (e.g. neck) marked, i.e. if there are multiple people in the extracted image patch there might be multiple correct keypoint positions. Only at step 2 and later they reduce that to the expected person (i.e. one keypoint position per map).

* Results
  * Example results:
    * ![Example results](images/Convolutional_Pose_Machines__results.jpg?raw=true "Example results")
  * Self-correction of predictions over several timesteps:
    * ![Effect of timesteps](images/Convolutional_Pose_Machines__timesteps.jpg?raw=true "Effect of timesteps")
  * They beat existing methods on the datasets MPII, LSP and FLIC.
  * Applying a loss function after each step (instead of only once after the last step) improved their results and reduced problems related to vanishing gradients.
  * The effective receptive field size of each step had a significant influence on the results. They increased it to up to 300px (about 80% of the image size) and saw continuous improvements in accuracy.
    * ![Receptive field size effect](images/Convolutional_Pose_Machines__rf_size.jpg?raw=true "Receptive field size effect")
