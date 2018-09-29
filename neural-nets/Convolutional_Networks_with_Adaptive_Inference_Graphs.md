# Paper

* **Title**: Convolutional Networks with Adaptive Inference Graphs
* **Authors**: Andreas Veit, Serge Belongie
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Andreas_Veit_Convolutional_Networks_with_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Andreas_Veit_Convolutional_Networks_with_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Gating, Gumbel, ECCV 2018
* **Year**: 2018

# Summary

* What
  * They introduce a dynamic gating mechanism for ResNet that decides on-the-fly which layers to execute for a given input image.
  * This improves performance, classification accuracy and robustness to adversarial attacks.
  * Note that the decision of whether to execute a layer during training is stochastic, making it closely related to stochastic depth.

* How
  * Their technique is based on residual connections, i.e. on ResNet.
  * They add a gating function, so that `x_l = x_{l-1} + F(x_{l-1})` becomes `x_l = x_{l-1} + gate(x_{l-1}) * F(x_{l-1})`.
  * Gating
    * The gating function produces a discrete output (`0` or `1`), enabling to completely skip a layer.
    * They implement the gating function by applying the following steps to an input feature map:
      1. Global average pooling, leads to `Cx1x1` output.
      2. Flatten to vector
      3. Apply fully connected layer with `d` outputs and ReLU
      4. Apply fully connected layer with `2` outputs
      5. Apply straight-through gumbel-softmax to output
    * Straight-through Gumbel-softmax is a standard technique that is comparable to softmax,
      but produces discrete `0`/`1` outputs during the forward pass and uses a softmax during the backward pass to approximate gradients.
    * For small `d`, the additional execution time of the gating function is negligible.
    * Visualization:
      * ![gating unit](images/Convolutional_Networks_with_Adaptive_Inference_Graphs/gating_unit.jpg?raw=true "gating unit")
  * They also introduce a target rate loss, which pushes the network to execute `t` percent of all layers (i.e. let `t` percent of all gates produce `1` as the output):
    * ![target loss](images/Convolutional_Networks_with_Adaptive_Inference_Graphs/target_loss.jpg?raw=true "target loss")
    * where `z_l` is the fraction of gates that produced `1` for layer `l` within the minibatch
    * and `t` is the target rate.
    * Note that this will probably cause problems for small minibatches.

* Results
  * CIFAR-10
    * They choose `d`=16, adds 0.01% FLOPS and 4.8% parameters to ResNet-110.
    * At `t`=0.7 they observe that on average 82% of all layers are executed.
      * The model chooses to always execute downsampling layers. They seem to be important.
    * When always executing all layers and scaling their outputs with their likelihood of being executed (similar to Dropout, Stochastic Depth),
      they achieve higher score than a network trained with stochastic depth. This indicates that their results do not just come from a regularizing effect.
    * ![cifar 10 scores](images/Convolutional_Networks_with_Adaptive_Inference_Graphs/cifar_10_scores.jpg?raw=true "cifar 10 scores")
  * ImageNet
    * They compare gated ResNet-50 and ResNet-110.
    * In ResNet-110 they always execute the first couple of layers (ungated), in ResNet-50 they gate all layers.
    * Trade-off between FLOPs and accuracy (ResNet-50/101 vs their model "ConvNet-AIG" at different `t`):
      * ![imagenet tradeoff](images/Convolutional_Networks_with_Adaptive_Inference_Graphs/imagenet_tradeoff.jpg?raw=true "imagenet tradeoff")
      * Their model becomes more accurate as `t` is lowered down from 1.0 to 0.7. After that the accuracy starts to drop.
    * Visualization of layerwise execution rate:
      * ![imagenet layerwise execution rate](images/Convolutional_Networks_with_Adaptive_Inference_Graphs/imagenet_layerwise_execution_rate.jpg?raw=true "imagenet layerwise execution rate")
      * The model learns to always execute downsampling layers.
      * The model seems to treat all man-made objects and all animals as two different groups with similar layers.
    * Execution distribution and execution frequency during training:
      * ![imagenet distribution](images/Convolutional_Networks_with_Adaptive_Inference_Graphs/imagenet_distribution.jpg?raw=true "imagenet distribution")
      * The model quickly learns to always execute the last layer.
    * The model requires especially many layers to classify non-iconic views of objects (e.g. dog face from unusual perspective instead of whole dog from the side).
    * When running adversarial attacks on ResNet-50 and their ConvNet-AIG-50 (via Fast Gradient Sign Attack) they observe that:
      1. Their model is overall more robust to adversarial attacks, i.e. keeps higher accuracy (both with/without protection via JPEG compression).
      2. The executed layers remain mostly the same in their model. They seem to not get attacked.

