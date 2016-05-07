# Paper

* **Title**: Let there be Color!: Joint End-to-end Learning of Global and Local Image Priors for Automatic Image Colorization with Simultaneous Classification
* **Authors**: Satoshi Iizuka, Edgar Simo-Serra, Hiroshi Ishikawa
* **Link**: http://hi.cs.waseda.ac.jp/~iizuka/projects/colorization/en/
* **Tags**: Neural Network, colorization
* **Year**: 2016

# Summary

![Algorithm](images/Playing_Atari_with_Deep_Reinforcement_Learning__algorithm.png?raw=true "Algorithm")

*The original full algorithm, as shown in the paper.*


--------------------

# Rough chapter-wise notes

* (1) Introduction
  * They use a CNN to color images.
  * Their network extracts global priors and local features from grayscale images.
  * Global priors:
    * Extracted from the whole image (e.g. time of day, indoor or outdoors, ...).
    * They use class labels of images to train those. (Not needed during test.)
  * Local features: Extracted from small patches (e.g. texture).
  * They don't generate a full RGB image, instead they generate the chrominance map using the CIE L\*a\*b colorspace.
  * Components of the model:
    * Low level features network: Generated first.
    * Mid level features network: Generated based on the low level features.
    * Global features network: Generated based on the low level features.
    * Colorization network: Receives mid level and global features, which were merged in a fusion layer.
  * Their network can process images of arbitrary size.
  * Global features can be generated based on another image to change the style of colorization, e.g. to change the seasonal colors from spring to summer.

* (3) Joint Global and Local Model
  * <repetition of parts of the introduction>
  * They mostly use ReLUs.
  * (3.1) Deep Networks
    * <standard neural net introduction>
  * (3.2) Fusing Global and Local Features for Colorization
    * Global features are used as priors for local features.
    * (3.2.1) Shared Low-Level Features
      * The low level features are which's (low level) features are fed into the networks of both the global and the medium level features extractors.
      * They generate them from the input image using a ConvNet with 6 layers (3x3, 1x1 padding, strided/no pooling, ends in 512xH/8xW/8).
    * (3.2.2) Global Image Features
      * They process the low level features via another network into global features.
      * That network has 4 conv-layers (3x3, 2 strided layers, all 512 filters), followed by 3 fully connected layers (1024, 512, 256).
      * Input size (of low level features) is expected to be 224x224.
    * (3.2.3) Mid-Level Features
      * Takes the low level features (512xH/8xW/8) and uses 2 conv layers (3x3) to transform them to 256xH/8xW/8.
    * (3.2.4) Fusing Global and Local Features
      * The Fusion Layer is basically an extended convolutional layer.
      * It takes the mid-level features (256xH/8xW/8) and the global features (256) as input and outputs a matrix of shape 256xH/8xW/8.
      * It mostly operates like a normal convolutional layer on the mid-level features. However, its weight matrix is extended to also include weights for the global features (which will be added at every pixel).
      * So they use something like `fusion at pixel u,v = sigmoid(bias + weights * [global features, mid-level features at pixel u,v])` - and that with 256 different weight matrices and biases for 256 filters.
    * (3.2.5) Colorization Network
      * The colorization network receives the 256xH/8xW/8 matrix from the fusion layer and transforms it to the 2xHxW chrominance map.
      * It basically uses two upsampling blocks, each starting with a nearest neighbour upsampling layer, followed by 2 3x3 convs.
      * The last layer uses a sigmoid activation.
      * The network ends in a MSE.
  * (3.3) Colorization with Classification
    * To make training more effective, they train parts of the global features network via image class labels.
    * I.e. they take the output of the 2nd fully connected layer (at the end of the global network), add one small hidden layer after it, followed by a sigmoid output layer (size equals number of class labels).
    * They train that with cross entropy. So their global loss becomes something like `L = MSE(color accuracy) + alpha*CrossEntropy(class labels accuracy)`.
  * (3.4) Optimization and Learning
    * 
