# Paper

* **Title**: Rainbow: Combining Improvements in Deep Reinforcement Learning
* **Authors**: Matteo Hessel, Joseph Modayil, Hado van Hasselt, Tom Schaul, Georg Ostrovski, Will Dabney, Dan Horgan, Bilal Piot, Mohammad Azar, David Silver
* **Link**: https://arxiv.org/abs/1710.02298
* **Tags**: Neural Network, reinforcement
* **Year**: 2017

# Summary

* What
  * They combine several previous improvements for reinforcement learning to one algorithm.
  * The combination beats previous methods by a good margin.
  * They analyze which of the used improvements has most influence on the result.

* How
  * They use the following improvements:
    * **Double Q-learning**
      * Uses two networks during training.
      * One predicts Q-values, the other is updated.
      * Usually done by using a copy with freezed parameters.
    * **Prioritized Replay**
      * Samples experiences from the replay memory based on the difference between predicted and real Q-values.
      * Recent experiences also get higher priority.
    * **Dueling Networks**
      * Splits the Q-value prediction into a value stream (mean value over all values) and an advantage stream (advantage of specific actions over others).
    * **Multi-Step Learning**
      * Splits `direct reward + Q(next state, next action)` into `direct reward + direct reward of next N actions + Q(next Nth state, next Nth action)` (weighted with discrount factor).
      * I.e. per training example, some experiences from the future are directly used to compute the rewards and only after some point the Q-function is used.
    * **Distributional RL**
      * Seems to be nothing else but switching the regression (of Q-values) to classification in order to avoid standard mode problems with regression.
      * The range of possible reward values is partitioned into N bins.
    * **Noisy Nets**
      * Gets rid of the exploration factor in epsilon-greedy strategies.
      * Instead it uses a noisy fully connected layer where the noise weights are learned by the network.
      * As the network becomes more accurate at predicting good Q-values, it automatically decreases the noise.
  * They combine all of the mentioned methods.
  * They use a KL term for weighting in Prioritized Replay (to account for the Distributional RL).
  * Training
    * They start training after 80k frames.
    * They use Adam.
    * When using epsilon-greedy instead of noise nets, they anneal epsilon to 0.01 at 250k frames.
    * For multi-step learning they use n=3 future experiences.

* Results
  * They evaluate Rainbow 57 Atari games.
  * Rainbow beats all other methods used on their own, both in learning speed and maximum skill level.
  * It performs far better than the classic DQN approach.
  * Average performance:
    * ![average performance](images/Rainbow/average_performance.jpg?raw=true "average performance")
  * Ablation
    * Removing the priority replay, multi-step learning or distributional RL significantly worsens the performance.
    * Removing noise nets also harms the performance, although a bit less.
    * Removing double Q-learning or dueling networks seems to have no significiant effect.
    * Visualization:
      * ![ablation](images/Rainbow/ablation.jpg?raw=true "ablation")
  * Learning curves by game:
    * ![by game](images/Rainbow/by_game.jpg?raw=true "by game")

