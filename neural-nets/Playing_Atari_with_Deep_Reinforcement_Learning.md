# Paper

* **Title**: Playing Atari with Deep Reinforcement Learning
* **Authors**: Volodymyr Mnih, Koray Kavukcuoglu, David Silver, Alex Graves, Ioannis Antonoglou, Daan Wierstra, Martin Riedmiller
* **Link**: http://arxiv.org/abs/1312.5602
* **Tags**: Neural Network, reinforcement learning
* **Year**: 2013

# Summary



![Generated Faces](images/Generative_Adversarial_Networks__faces.jpg?raw=true "Generated Faces")

--------------------

# Rough chapter-wise notes

* (1) Introduction
  * Problems when using neural nets in reinforcement learning (RL):
    * Reward signal is often sparse, noise and delayed.
    * Often assumption that data samples are independent, while they are correlated in RL.
    * Data distribution can change when the algorithm learns new behaviours.
  * They use Q-learning with a CNN and stochastic gradient descent.
  * They use an experience replay mechanism (i.e. memory) from which they can sample previous transitions (for training).
  * They apply their method to Atari 2600 games in the Arcade Learning Environment (ALE).
  * They use only the visible pixels as input to the network, i.e. no manual feature extraction.

* (2) Background

* (3) Related Work

* (4) Deep Reinforcement Learning
  * They want to connect a reinforcement learning algorithm with a deep neural network, e.g. to get rid of handcrafted features.
  * The network is supposes to run on the raw RGB images.
  * They use experience replay, i.e. store tuples of (pixels, chosen action, received reward) in a memory and use that during training.
  * They use Q-learning.
  * They use an epsilon-greedy policy.
  * Advantages from using experience replay instead of learning "live" during game playing:
    * Experiences can be reused many times (more efficient).
    * Samples are less correlated.
    * Learned parameters from one batch don't determine as much the distributions of the examples in the next batch.
  * They save the last N experiences and sample uniformly from them during training.
  * (4.1) Preprocessing and Model Architecture
    * Raw Atari images are 210x160 pixels with 128 possible colors.
    * They downsample them to 110x84 pixels and then crop the 84x84 playing area out of them.
    * They also convert the images to grayscale.
    * They use the last 4 frames as input and stack them.
    * So their network input has shape 84x84x4.
    * They use one output neuron per possible action. So they can compute the Q-value (expected reward) of each action with one forward pass.
    * Architecture: 84x84x4 (input) => 16 8x8 convs, stride 4, ReLU => 32 4x4 convs stride 2 ReLU => fc 256, ReLU => fc N actions, linear
    * 4 to 18 actions/outputs (depends on the game).
    * Aside from the outputs, the architecture is the same for all games.

* (5) Experiments
  * Games that they played: Beam Rider, Breakout, Enduro, Pong, Qbert, Seaquest, Space Invaders
  * They use the same architecture und hyperparameters for all games.
  * They give a reward of +1 whenever the in-game score increases and -1 whenever it decreases.
  * They use RMSProp.
  * Mini batch size was 32.
  * They train for 10 million frames/examples.
  * They initialize epsilon (in their epsilon greedy strategy) to 1.0 and decrease it linearly to 0.1 at one million frames.
  * They let the agent decide upon an action at every 4th in-game frame (3rd in space invaders).
  * (5.1) Training and stability
    * They plot the average reward und Q-value per N games to evaluate the agent's training progress,
    * The average reward increases in a noisy way.
    * The average Q value increases smoothly.
    * They did not experience any divergence issues during their training.
