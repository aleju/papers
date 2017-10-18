# Paper

* **Title**: Learning to Navigate in Complex Environments
* **Authors**: Piotr Mirowski, Razvan Pascanu, Fabio Viola, Hubert Soyer, Andrew J. Ballard, Andrea Banino, Misha Denil, Ross Goroshin, Laurent Sifre, Koray Kavukcuoglu, Dharshan Kumaran, Raia Hadsell
* **Link**: https://arxiv.org/abs/1611.03673
* **Tags**: Neural Network, reinforcement, A3C
* **Year**: 2016

# Summary

* What
  * They describe a model based on the A3C algorithm.
  * The model is optimized so that it learns fast to navigate through mazes/games.
  * Most of the modification is based on adding auxiliary losses (depth prediction and "have I been here before?").

* How
  * Their model is based on the A3C algorithm and works mostly in the same way.
  * They describe four variations of the model:
    * **FF A3C**: Standard A3C based agent with a simple CNN.
    * **LSTM A3C**: Standard A3C based agent with a simple CNN+LSTM.
    * **Nav A3C**: Like "LSTM A3C", but uses two stacked LSTMs and gets three additional inputs: last received reward, last performed action, current velocity vector (lateral + rotation).
    * **Nav A3C +D1D2L**: Like "Nav A3C", but additionally predicts *depth information* and *loop information* (see below).
    * Visualizations:
      * ![models](images/Learning_to_Navigate_in_Complex_Environments/models.jpg?raw=true "models")
  * All of these models predict (at least) a policy (`pi(a_t|s_t)`) and value function (`V(s_t)`).
  * Depth prediction
    * They let the model predict the depth map of the currently visible scene/screen.
    * They could also feed the depth map as an additional input into the model,
      but argue that it helps more with understanding and learning the navigation task to let the model predict it.
    * They try prediction both via regression and classification.
      (Where classification uses non-uniform bins, so that there are more bins for far away depths.)
  * Loop closure prediction
    * They let the model predict whether it
      * has been at (roughly) the current position `t` timesteps in the past
      * AND also moved sufficiently far away from that position between now and `t` timesteps in the past (in order to not make the loss too simple)

* Results
  * They use a maze game from DeepMind Labs for training and testing.
  * In that maze, the agent can accelerate forward/backward/sideways/rotational.
  * The agent must find a goal within a short amount of time to gain 10 reward. Occasionally there are other reward objects giving +1 or +2.
  * They use the following maze types:
  * **Static maze**: Goal locations are always the same, but the agent is placed at a random location after finding them.
  * **Dynamic maze / random goal maze**: Goal locations vary per episode.
  * **I-Maze**: Static maze layout. Goal is at a random one of four locations per episode.
  * Observations:
    * Predicting depth via regression works significantly worse than via classification.
    * FF A3C (raw feed-forward CNN without RNN-memory) can sometimes still navigate quite well.
      Memory is apparently not always required for navigation, despite ambiguity.
    * Adding the last action, last reward and velocity as an input seems to usually help the agent, but not always.
    * Adding the auxiliary losses helps significantly with learning.
  * Learning curves:
    * ![learning curves](images/Learning_to_Navigate_in_Complex_Environments/learning_curves.jpg?raw=true "learning curves")
    * Legend: `-L` = with loop closure prediction, `-D1` = with depth prediction after first LSTM layer, `-D2` = with depth prediction after second LSTM layer.

