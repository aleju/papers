# Paper

* **Title**: End to End Learning for Self-Driving Cars
* **Authors**: Mariusz Bojarski, Davide Del Testa, Daniel Dworakowski, Bernhard Firner, Beat Flepp, Prasoon Goyal, Lawrence D. Jackel, Mathew Monfort, Urs Muller, Jiakai Zhang, Xin Zhang, Jake Zhao, Karol Zieba
* **Link**: https://arxiv.org/abs/1604.07316
* **Tags**: Neural Network, self-driving
* **Year**: 2016

# Summary

* What
  * They describe a model that automatically steers cars on roads.
  * The model runs in realtime.
  * The model is trained from images and corresponding (correct) steering wheel angles.

* How
  * Architecture
    * Their model is just a standard CNN with a few convolutional and fully connected layers.
    * They have a single output: the correct steering wheel angle predicted by the model.
      (The model does not predict whether to accelerate or brake.)
      * To be more precise: The steering wheel angle is predicted as `1/r`, where `r` is the turning radius of the car.
        This way, the prediction is more independent of the used car.
        They use `1/r` instead of `r` in order to avoid getting infinity when driving straight.
    * Their input images are in YUV color space.
    * They use a hard coded (not learned) normalization layer (no further explanations in paper).
    * Visualization:
      * ![Architecture](images/End_to_End_Learning_for_Self-Driving_Cars/architecture.jpg?raw=true "Architecture")
  * Data collection
    * They drive on roads with cars, collecting photos and corresponding steering wheel angles.
    * They drive on different road settings (e.g. highway, tunnels, residential roads) and weather (sunny, cloudy, foggy, ...).
    * They collected about 72 hours of driving.
    * They annotate each example with the road center location, road type and lane switching information ("stays in lane"/"is switching lanes").
  * Data augmentation
    * Training on just the collected images is not going to work, because they only show correct driving.
      Sooner or later the model would make a mistake and end up going slightly off road.
      This would then be a situation that it has never seen before and the predicted output becomes more or less random, leading to crashes.
    * They attached two more cameras to the cars during data collection, one pointing to the left and one to the right.
    * They use these images as examples of bad situations and change the ground truth steering wheel angle
      so that it would steer the car back onto the road within 2 seconds.
    * They also generate additional images between the center and left/right cameras.
      They do this by viewpoint transformation and assume that all point above the horizon line are infinitely far away,
      while all below the horizon line are on flat grounds.
      (No further explanation here on how they transform images and annotations *exactly*. And where do they get the horizon line from?)
    * They also seem to apply random rotations and translations to the images (or maybe that is meant by viewpoint transformations?).
  * Training
    * They only train on examples where the driver stays in a lane (as opposed to switching lanes).
    * They subsample the input at 10fps (collection was at 30fps) in order to not get too similar examples.
    * Training and application happens in Torch7.
    * The system can predict at 30fps.

* Results
  * Simulation
    * They simulate on the collected data how the model would steer.
    * I.e. they predict steering wheel angles and then transform future images as if that angle had been chosen.
    * They measure how far the simulated car would be away from the road center.
    * They measure an virtual, human intervention (of 6 seconds) whenever the car gets too far away from the center.
    * They can then estimate the simulated time that the car would have been autonomous.
    * The actual number though is nowhere in the paper. x(
  * They drive in Monmouth County, being autonomous roughly 98% of the time.
  * They drive 16.09km on Garden State Parkway, being autonomous 100% of the time.

