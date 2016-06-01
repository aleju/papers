# Paper

* **Title**: Rank Ordered Autoencoders
* **Authors**: Paul Bertens
* **Link**: https://arxiv.org/abs/1605.01749
* **Tags**: Neural Network, autoencoder, rank order
* **Year**: 2016

# Summary

* What
  * Autoencoders typically have some additional criterion that pushes them towards learning meaningful representations.
    * E.g. L1-Penalty on the code layer (z), Dropout on z, Noise on z.
    * Often, representations with sparse activations are considered meaningful (so that each activation reflects are clear concept).
  * This paper introduces another technique that leads to sparsity.
  * They use a rank ordering on z.
  * The first (according to the ranking) activations have to do most of the reconstruction work of the data (i.e. image).

* How
  * Basic architecture:
    * They use an Autoencoder architecture: Input -> Encoder -> z -> Decoder -> Output.
    * Their encoder and decoder seem to be empty, i.e. z is the only hidden layer in the network.
    * Their output is not just one image (or whatever is encoded), instead they generate one for every unit in layer z.
    * Then they order these outputs based on the activation of the units in z (rank ordering), i.e. the output of the unit with the highest activation is placed in the first position, the output of the unit with the 2nd highest activation gets the 2nd position and so on.
    * They then generate the final output image based on a cumulative sum. So for three reconstructed output images `I1, I2, I3` (rank ordered that way) they would compute `I1 + (I1+I2) + (I1+I2+I3)`.
    * They then compute the error based on that reconstruction (`reconstruction - input image`) and backpropagate it.
  * 

* Results

![Examples](images/Accurate_Image_Super-Resolution__examples.png?raw=true "Examples")

*Super-resolution quality of their model (top, bottom is a competing model).*
