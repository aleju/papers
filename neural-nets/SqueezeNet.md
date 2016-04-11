# Paper

* **Title**: SqueezeNet: AlexNet-level accuracy with 50x fewer parameters and <0.5MB model size
* **Authors**: Forrest N. Iandola, Song Han, Matthew W. Moskewicz, Khalid Ashraf, William J. Dally, Kurt Keutzer
* **Link**: http://arxiv.org/abs/1602.07360v3
* **Tags**: Neural Network, compression, AlexNet, residual
* **Year**: 2016

# Summary

* What
  * The authors train a variant of AlexNet that has significantly fewer parameters than the original network, while keeping the network's accuracy stable.
  * Advantages of this:
    * More efficient distributed training, because less parameters have to be transferred.
    * More efficient transfer via the internet, because the model's file size is smaller.
    * Possibly less memory demand in production, because fewer parameters have to be kept in memory.

* How
  * They define a Fire Module. A Fire Module contains of:
    * Squeeze Module: A 1x1 convolution that reduces the number of channels (e.g. from 128x32x32 to 64x32x32).
    * Expand Module: A 1x1 convolution and a 3x3 convolution, both applied to the output of the Squeeze Module. Their results are concatenated.
  * Using many 1x1 convolutions is advantageous, because they need less parameters than 3x3s.
  * They use ReLUs, only convolutions (no fully connected layers) and Dropout (50%, before the last convolution).
  * They use late maxpooling. They argue that applying pooling late - rather than early - improves accuracy while not needing more parameters.
  * They try residual connections:
    * One network without any residual connections (performed the worst).
    * One network with residual connections based on identity functions, but only between layers of same dimensionality (performed the best).
    * One network with residual connections based on identity functions and other residual connections with 1x1 convs (where dimensionality changed) (performance between the other two).
  * They use pruning from Deep Compression to reduce the parameters further. Pruning simply collects the 50% of all parameters of a layer that have the lowest values and sets them to zero. That creates a sparse matrix.

* Results
  * 50x parameter reduction of AlexNet (1.2M parameters before pruning, 0.4M after pruning).
  * 510x file size reduction of AlexNet (from 250mb to 0.47mb) when combined with Deep Compression.
  * Top-1 accuracy remained stable.
  * Pruning apparently can be used safely, even after the network parameters have already been reduced significantly.
  * While pruning was generally safe, they found that two of their later layers reacted quite sensitive to it. Adding parameters to these (instead of removing them) actually significantly improved accuracy.
  * Generally they found 1x1 convs to react more sensitive to pruning than 3x3s. Therefore they focused pruning on 3x3 convs.
  * First pruning a network, then re-adding the pruned weights (initialized with 0s) and then retraining for some time significantly improved accuracy.
  * The network was rather resilient to significant channel reduction in the Squeeze Modules. Reducing to 25-50% of the original channels (e.g. 128x32x32 to 64x32x32) seemed to be a good choice.
  * The network was rather resilient to removing 3x3 convs and replacing them with 1x1 convs. A ratio of 2:1 to 1:1 (1x1 to 3x3) seemed to produce good results while mostly keeping the accuracy.
  * Adding some residual connections between the Fire Modules improved the accuracy.
  * Adding residual connections with identity functions *and also* residual connections with 1x1 convs (where dimensionality changed) improved the accuracy, but not as much as using *only* residual connections with identity functions (i.e. it's better to keep some modules without identity functions).


--------------------

# Rough chapter-wise notes

* (1) Introduction and Motivation
  * Advantages from having less parameters:
    * More efficient distributed training, because less data (parameters) have to be transfered.
    * Less data to transfer to clients, e.g. when a model used by some app is updated.
    * FPGAs often have hardly any memory, i.e. a model has to be small to be executed.
  * Target here: Find a CNN architecture with less parameters than an existing one but comparable accuracy.

* (2) Related Work
  * (2.1) Model Compression
    * SVD-method: Just apply SVD to the parameters of an existing model.
    * Network Pruning: Replace parameters below threshold with zeros (-> sparse matrix), then retrain a bit.
    * Add quantization and huffman encoding to network pruning = Deep Compression.
  * (2.2) CNN Microarchitecture
    * The term "CNN Microarchitecture" refers to the "organization and dimensions of the individual modules" (so an Inception module would have a complex CNN microarchitecture).
  * (2.3) CNN Macroarchitecture
    * CNN Macroarchitecture = "big picture" / organization of many modules in a network / general characteristics of the network, like depth
    * Adding connections between modules can help (e.g. residual networks)
  * (2.4) Neural Network Design Space Exploration
    * Approaches for Design Space Exporation (DSE):
      * Bayesian Optimization, Simulated Annealing, Randomized Search, Genetic Algorithms

* (3) SqueezeNet: preserving accuracy with few parameters
  * (3.1) Architectural Design Strategies
    * A conv layer with N filters applied to CxHxW input (e.g. 3x128x128 for a possible first layer) with kernel size kHxkW (e.g. 3x3) has `N*C*kH*kW` parameters.
    * So one way to reduce the parameters is to decrease kH and kW, e.g. from 3x3 to 1x1 (reduces parameters by a factor of 9).
    * A second way is to reduce the number of channels (C), e.g. by using 1x1 convs before the 3x3 ones.
    * They think that accuracy can be improved by performing downsampling later in the network (if parameter count is kept constant).
  * (3.2) The Fire Module
    * The Fire Module has two components:
      * Squeeze Module:
        * One layer of 1x1 convs
      * Expand Module:
        * Concat the results of:
          * One layer of 1x1 convs
          * One layer of 3x3 convs
    * The Squeeze Module decreases the number of input channels significantly.
    * The Expand Module then increases the number of input channels again.
  * (3.3) The SqueezeNet architecture
    * One standalone conv, then several fire modules, then a standalone conv, then global average pooling, then softmax.
    * Three late max pooling laters.
    * Gradual increase of filter numbers.
    * (3.3.1) Other SqueezeNet details
      * ReLU activations
      * Dropout before the last conv layer.
      * No linear layers.

* (4) Evaluation of SqueezeNet
  * Results of competing methods:
    * SVD: 5x compression, 56% top-1 accuracy
    * Pruning: 9x compression, 57.2% top-1 accuracy
    * Deep Compression: 35x compression, ~57% top-1 accuracy
  * SqueezeNet: 50x compression, ~57% top-1 accuracy
  * SqueezeNet combines low parameter counts with Deep Compression.
  * The accuracy does not go down because of that, i.e. apparently Deep Compression can even be applied to small models without giving up on performance.

* (5) CNN Microarchitecture Design Space Exploration
  * (5.1) CNN Microarchitecture metaparameters
    * blabla we test various values for this and that parameter
  * (5.2) Squeeze Ratio
    * In a Fire Module there is first a Squeeze Module and then an Expand Module. The Squeeze Module decreases the number of input channels to which 1x1 and 3x3 both are applied (at the same time).
    * They analyzed how far you can go down with the Sqeeze Module by training multiple networks and calculating the top-5 accuracy for each of them.
    * The accuracy by Squeeze Ratio (percentage of input channels kept in 1x1 squeeze, i.e. 50% = reduced by half, e.g. from 128 to 64):
      * 12%: ~80% top-5 accuracy
      * 25%: ~82% top-5 accuracy
      * 50%: ~85% top-5 accuracy
      * 75%: ~86% top-5 accuracy
      * 100%: ~86% top-5 accuracy
  * (5.3) Trading off 1x1 and 3x3 filters
    * Similar to the Squeeze Ratio, they analyze the optimal ratio of 1x1 filters to 3x3 filters.
    * E.g. 50% would mean that half of all filters in each Fire Module are 1x1 filters.
    * Results:
      * 01%: ~76% top-5 accuracy
      * 12%: ~80% top-5 accuracy
      * 25%: ~82% top-5 accuracy
      * 50%: ~85% top-5 accuracy
      * 75%: ~85% top-5 accuracy
      * 99%: ~85% top-5 accuracy

* (6) CNN Macroarchitecture Design Space Exploration
  * They compare the following networks:
    * (1) Without residual connections
    * (2) With residual connections between modules of same dimensionality
    * (3) With residual connections between all modules (except pooling layers) using 1x1 convs (instead of identity functions) where needed
  * Adding residual connections (2) improved top-1 accuracy from 57.5% to 60.4% without any new parameters.
  * Adding complex residual connections (3) worsed top-1 accuracy again to 58.8%, while adding new parameters.

* (7) Model Compression Design Space Exploration
  * (7.1) Sensitivity Analysis: Where to Prune or Add parameters
    * They went through all layers (including each one in the Fire Modules).
    * In each layer they set the 50% smallest weights to zero (pruning) and measured the effect on the top-5 accuracy.
    * It turns out that doing that has basically no influence on the top-5 accuracy in most layers.
    * Two layers towards the end however had significant influence (accuracy went down by 5-10%).
    * Adding parameters to these layers improved top-1 accuracy from 57.5% to 59.5%.
    * Generally they found 1x1 layers to be more sensitive than 3x3 layers so they pruned them less aggressively.
  * (7.2) Improving Accuracy by Densifying Sparse Models
    * They found that first pruning a model and then retraining it again (initializing the pruned weights to 0) leads to higher accuracy.
    * They could improve top-1 accuracy by 4.3% in this way.
