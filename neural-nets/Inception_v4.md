# Paper

* **Title**: Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning
* **Authors**: Christian Szegedy, Sergey Ioffe, Vincent Vanhoucke
* **Link**: http://arxiv.org/abs/1602.07261v1
* **Tags**: Neural Network, Inception, Residual
* **Year**: 2016

# Summary

* Overview
  * Inception v4 is like Inception v3, but
    * Slimmed down, i.e. some parts were simplified
    * One new version with residual connections (Inception-ResNet-v2), one without (Inception-v4)
  * They didn't observe an improved error rate when using residual connections.
  * They did however oberserve that using residual connections decreased their training times.
  * They had to scale down the results of their residual modules (multiply them by a constant ~0.1). Otherwise their networks would die (only produce 0s).
  * Results on ILSVRC 2012 (val set, 144 crops/image):
    * Top-1 Error:
      * Inception-v4: 17.7%
      * Inception-ResNet-v2: 17.8%
    * Top-5 Error (ILSVRC 2012 val set, 144 crops/image):
      * Inception-v4: 3.8%
      * Inception-ResNet-v2: 3.7% 

* Architecture
  * Basic structure of Inception-ResNet-v2 (layers, dimensions):
    * `Image -> Stem -> 5x Module A -> Reduction-A -> 10x Module B -> Reduction B -> 5x Module C -> AveragePooling -> Droput 20% -> Linear, Softmax`
    * `299x299x3 -> 35x35x256 -> 35x35x256 -> 17x17x896 -> 17x17x896 -> 8x8x1792 -> 8x8x1792 -> 1792 -> 1792 -> 1000`
  * Modules A, B, C are very similar.
  * They contain 2 (B, C) or 3 (A) branches.
  * Each branch starts with a 1x1 convolution on the input.
  * All branches merge into one 1x1 convolution (which is then added to the original input, as usually in residual architectures).
  * Module A uses 3x3 convolutions, B 7x1 and 1x7, C 3x1 and 1x3.
  * The reduction modules also contain multiple branches. One has max pooling (3x3 stride 2), the other branches end in convolutions with stride 2.

![Module A](images/Inception_v4__module_a.png?raw=true "Module A")
![Module B](images/Inception_v4__module_b.png?raw=true "Module B")
![Module C](images/Inception_v4__module_c.png?raw=true "Module C")
![Reduction Module A](images/Inception_v4__reduction_a.png?raw=true "Reduction Module A")

*From top to bottom: Module A, Module B, Module C, Reduction Module A.*

![Top 5 error](images/Inception_v4__top5_error.png?raw=true "Top 5 error")

*Top 5 eror by epoch, models with (red, solid, bottom) and without (green, dashed) residual connections.*

-------------------------

# Rough chapter-wise notes

* Introduction, Related Work
  * Inception v3 was adapted to run on DistBelief. Inception v4 is designed for TensorFlow, which gets rid of some constraints and allows a simplified architecture.
  * Authors don't think that residual connections are inherently needed to train deep nets, but they do speed up the training.
  * History:
    * Inception v1 - Introduced inception blocks
    * Inception v2 - Added Batch Normalization
    * Inception v3 - Factorized the inception blocks further (more submodules)
    * Inception v4 - Adds residual connections

* Architectural Choices
  * Previous architectures were constrained due to memory problems. TensorFlow got rid of that problem.
  * Previous architectures were carefully/conservatively extended. Architectures ended up being quite complicated. This version slims down everything.
  * They had problems with residual networks dieing when they contained more than 1000 filters (per inception module apparently?). They could fix that by multiplying the results of the residual subnetwork (before the element-wise addition) with a constant factor of ~0.1.

* Training methodology
  * Kepler GPUs, TensorFlow, RMSProb (SGD+Momentum apprently performed worse)

* Experimental Results
  * Their residual version of Inception v4 ("Inception-ResNet-v2") seemed to learn faster than the non-residual version.
  * They both peaked out at almost the same value.
  * Top-1 Error (ILSVRC 2012 val set, 144 crops/image):
    * Inception-v4: 17.7%
    * Inception-ResNet-v2: 17.8%
  * Top-5 Error (ILSVRC 2012 val set, 144 crops/image):
    * Inception-v4: 3.8%
    * Inception-ResNet-v2: 3.7%
