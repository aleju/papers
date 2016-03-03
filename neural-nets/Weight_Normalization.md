# Paper

* **Title**: Weight Normalization: A Simple Reparameterization to Accelerate Training of Deep Neural Networks
* **Authors**: Tim Salimans, Diederik P. Kingma
* **Link**: http://arxiv.org/abs/1602.07868
* **Tags**: Neural Network, Normalization, VAE
* **Year**: 2016

# Summary

* What it is
  * Weight Normalization (WN) is a normalization technique, similar to Batch Normalization (BN).
  * It normalizes each layer's weights.

* Differences to BN
  * WN normalizes based on each weight vector's orientation and magnitude. BN normalizes based on each weight's mean and variance in a batch.
  * WN works on each example on its own. BN works on whole batches.
  * WN is more deterministic than BN (due to not working an batches).
  * WN is better suited for noisy environment (RNNs, LSTMs, reinforcement learning, generative models). (Due to being more deterministic.)
  * WN is computationally simpler than BN.

* How its done
  * WN is a module added on top of a linear or convolutional layer.
  * If that layer's weights are `w` then WN learns two parameters `g` (scalar) and `v` (vector, identical dimension to `w`) so that `w = gv / ||v||` is fullfilled (`||v||` = euclidean norm of v).
  * `g` is the magnitude of the weights, `v` are their orientation.
  * `v` is initialized to zero mean and a standard deviation of 0.05.
  * For networks without recursions (i.e. not RNN/LSTM/GRU):
    * Right after initialization, they feed a single batch through the network.
    * For each neuron/weight, they calculate the mean and standard deviation after the WN layer.
    * They then adjust the bias to `-mean/stdDev` and `g` to `1/stdDev`.
    * That makes the network start with each feature being roughly zero-mean and unit-variance.
    * The same method can also be applied to networks without WN.

* Results:
  * They define BN-MEAN as a variant of BN which only normalizes to zero-mean (not unit-variance).
  * CIFAR-10 image classification (no data augmentation, some dropout, some white noise):
    * WN, BN, BN-MEAN all learn similarly fast. Network without normalization learns slower, but catches up towards the end.
    * BN learns "more" per example, but is about 16% slower (time-wise) than WN.
    * WN reaches about same test error as no normalization (both ~8.4%), BN achieves better results (~8.0%).
    * WN + BN-MEAN achieves best results with 7.31%.
    * Optimizer: Adam
  * Convolutional VAE on MNIST and CIFAR-10:
    * WN learns more per example und plateaus at better values than network without normalization. (BN was not tested.)
    * Optimizer: Adamax
  * DRAW on MNIST (heavy on LSTMs):
    * WN learns significantly more example than network without normalization.
    * Also ends up with better results. (Normal network might catch up though if run longer.)
  * Deep Reinforcement Learning (Space Invaders):
    * WN seemed to overall acquire a bit more reward per epoch than network without normalization. Variance (in acquired reward) however also grew.
    * Results not as clear as in DRAW.
    * Optimizer: Adamax

* Extensions
  * They argue that initializing `g` to `exp(cs)` (`c` constant, `s` learned) might be better, but they didn't get better test results with that.
  * Due to some gradient effects, `||v||` currently grows monotonically with every weight update. (Not necessarily when using optimizers that use separate learning rates per parameters.)
  * That grow effect leads the network to be more robust to different learning rates.
  * Setting a small hard limit/constraint for `||v||` can lead to better test set performance (parameter updates are larger, introducing more noise).


![CIFAR-10 results](images/Weight_Normalization__cifar10.png?raw=true "CIFAR-10 results")

*Performance of WN on CIFAR-10 compared to BN, BN-MEAN and no normalization.*

![DRAW, DQN results](images/Weight_Normalization__draw_dqn.png?raw=true "DRAW, DQN results")

*Performance of WN for DRAW (left) and deep reinforcement learning (right).*
