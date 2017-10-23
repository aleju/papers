# Paper

* **Title**: Virtual to Real Reinforcement Learning for Autonomous Driving
* **Authors**: Xinlei Pan, Yurong You, Ziyan Wang, Cewu Lu
* **Link**: https://arxiv.org/abs/1704.03952
* **Tags**: Neural Network, self-driving, reinforcement
* **Year**: 2017

# Summary

* What
  * They suggest a method to train a model for self-driving cars in a (virtual) game, while letting each frame look like a real one.
  * This allows to learn driving policies via reinforcement learning in a virtual environment, yet (later on) use them in real cars.

* How
  * Basics
    * The method is based on GANs, similar to something like CycleGAN.
    * The basic idea is to transform each game image/frame into a (semantic) segmentation map.
      Then the segmentation map is transformed into a realistic looking image.
      Both steps use GANs.
    * The model is then trained on the realistic looking images, instead of the game frames.
      (Why not just train it on the segmentation maps...?)
    * They argue that the segmentation maps can be viewed as the semantic representation between both (fake & real) images.
      (Similar to how machine translation models often convert each sentence to a vector representing the semantics before generating the translated sentence.)
    * Visualization of the architecture:
      * ![Architecture](images/Virtual_to_Real_RL_for_AD/architecture.jpg?raw=true "Architecture")
  * Loss
    * They use conditional GANs.
    * The generator gets the frame image `x` and a noise vector `z` and has to generate a segmentation map `s`.
    * The discriminator gets the frame image `x` and a segmentation map `s` and has to tell whether `s` is real or fake.
    * A second pair of generator and discriminator is then used to turn `s` into real images.
    * They use the standard GAN loss.
    * They add an L1 loss to "suppress blurring".
      (??? GANs shouldn't generate blurry images.
       This sounds more like they train G to predict `s` using the L1 loss.
       They then end up with blurry images, so they add the GAN loss to make them sharp.)
    * Full loss:
      * ![loss](images/Virtual_to_Real_RL_for_AD/loss.jpg?raw=true "loss")
  * Agent
    * They use the A3C algorithm for training. (12 threads)
    * Their reward function incentivizes fast speeds with the car being close to the road's center.
      They punish collisions.
    * Reward function:
      * ![reward](images/Virtual_to_Real_RL_for_AD/reward.jpg?raw=true "reward")
      * `v_t` is the speed in m/s
      * `alpha` is the angle in rad
      * `beta = 0.006`
      * `gamma = -0.025`
    * They predict 9 actions: left/straight/right, each with accelerate/brake/nothing.
  * Game
    * They train on the game "TORCS". (I guess that game provides segmentation maps for each game frame?)

* Results
  * They train their model in TORCS on track X and evaluate on Y.
    They achieve slightly better scores than a competing model trained on several tracks (A, B, C, D, ..., but not on X).
    A model trained directly on X peforms significantly better.
    * ![TORCS results](images/Virtual_to_Real_RL_for_AD/torcs_results.jpg?raw=true "TORCS results")
  * They test their model on a dataset associated with the NVIDIA self-driving paper.
    They reach 43% correct, while the supervised method reaches 53%.
    A competing model "B-RL" reaches 28% (reinforcement learned, but only on game images).
  * Example translations from game to real:
    * ![examples](images/Virtual_to_Real_RL_for_AD/examples.jpg?raw=true "examples")

