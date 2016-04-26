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
