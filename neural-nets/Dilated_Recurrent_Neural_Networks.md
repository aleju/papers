# Paper

* **Title**: Dilated Recurrent Neural Networks
* **Authors**: Shiyu Chang, Yang Zhang, Wei Han, Mo Yu, Xiaoxiao Guo, Wei Tan, Xiaodong Cui, Michael Witbrock, Mark Hasegawa-Johnson, Thomas Huang
* **Link**: https://arxiv.org/abs/1710.02224
* **Tags**: Neural Network, recurrent
* **Year**: 2017

# Summary

* What
  * They add a dilation factor to recurrent neural networks (e.g. LSTMs), similar to dilation in convolutions.
  * This enables learning of long-term dependencies, prevents vanishing/exploding gradients and makes the networks sometimes more parallelizable.

* How
  * Dilation
    * Dilation `d` here simply means, that each timestep gets input from the `d`-th previous timestep.
    * With `d=1`, this is identical to a normal reccurent network. With `d=2` there is a gap of 1 between each timestep.
    * They suggest to let the dilation exponentially increase per layer, e.g. 1, 2, 4, ...
    * Visualization:
      * ![dilation](images/Dilated_Recurrent_Neural_Networks/dilation.jpg?raw=true "dilation")
    * Using dilation can make it possible to execute some steps of the RNN in parallel.
      The following visualization shows that:
      * ![parallelization](images/Dilated_Recurrent_Neural_Networks/parallelization.jpg?raw=true "parallelization")
    * The dilation may start at a value higher than `1`.
      This however should be compensated before generating the output vector.
      To do that, output of the last layer at timestep `t` *and* `t-1` has to be used. Visualization:
      * ![minimum dilation](images/Dilated_Recurrent_Neural_Networks/minimum_dilation.jpg?raw=true "minimum dilation")
  * Memory capacity
    * They show that the memory capacity of dilated RNNs is better than in skip RNNs, i.e. the average path length in the network is shorter.

* Results
  * Copy-Task
    * This task involves copying of inputs.
    * I.e. the network gets some integers at the start, then `T={500, 1000}` timesteps pass, then it has to output the input values.
    * They use 9 layers with a dilation of up to 256. (This means that it is really almost raw copying of data for the network.)
    * Dilated RNNs perform best here, followed by dilated GRUs and dilated LSTMs.
    * All non-dilated networks resort to random guessing (i.e. fail to learn anything).
  * MNIST
    * They predict classes on MNIST, where each image is turned into a 784-element vector.
    * All networks, including competitors, can handle that task.
    * They make the task harder by padding the vectors to 1000 and 2000 elements length.
      Then only their dilated networks, dilated (1d-)CNNs and RNNs with skip connections can handle the task.
    * Their dilated RNNs learn faster the more layers they have.
      With 2 layers they learn nothing.
      With few layers they can sometimes also have major swings in accuracy during training.
      * ![mnist layers](images/Dilated_Recurrent_Neural_Networks/mnist_layers.jpg?raw=true "mnist layers")
    * When increasing the minimum dilation, they find that they can drop layers and still achieve almost the same accuracy,
      leading to much faster training (wall-clock time).
      Training with minimum dilation 2 leads to same accuracy at half the training time.
  * Language modelling
    * They test their models on Penn Treebank character predictions.
    * Their models get beaten by LayerNorm HM-LSTM, HyperNetworks, Zoneout.
    * They argue though that their models achieve the highest scores among models that do not use any normalization.
  * Speaker identification from raw waveform
    * They train on VCTK.
    * Their dilated models achieve much higher accuracy than non-dilated ones and can compete with models using MFCC features.

