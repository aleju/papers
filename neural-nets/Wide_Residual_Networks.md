# Paper

* **Title**: Wide Residual Networks
* **Authors**: Sergey Zagoruyko, Nikos Komodakis
* **Link**: http://arxiv.org/abs/1605.07146v1
* **Tags**: Neural Network, residual
* **Year**: 2016

# Summary

* What
  * The authors start with a standard ResNet architecture (i.e. residual network has suggested in "Identity Mappings in Deep Residual Networks").
    * Their residual block:
      ![Residual block](images/Wide_Residual_Networks__residual_block.png?raw=true "Residual block")
    * Several residual blocks of 16 filters per conv-layer, followed by 32 and then 64 filters per conv-layer.
  * They empirically try to answer the following questions:
    * How many residual blocks are optimal? (Depth)
    * How many filters should be used per convolutional layer? (Width)
    * How many convolutional layers should be used per residual block?
    * Does Dropout between the convolutional layers help?

* Results
  * *Layers per block and kernel sizes*:
    * Using 2 convolutional layers per residual block seems to perform best:
      ![Convs per block](images/Wide_Residual_Networks__convs_per_block.png?raw=true "Convs per block")
    * Using 3x3 kernel sizes for both layers seems to perform best.
    * However, using 3 layers with kernel sizes 3x3, 1x1, 3x3 and then using less residual blocks performs nearly as good and decreases the required time per batch.
  * *Width and depth*:
    * Increasing the width considerably improves the test error.
    * They achieve the best results (on CIFAR-10) when decreasing the depth to 28 convolutional layers, with each having 10 times their normal width (i.e. 16\*10 filters, 32\*10 and 64\*10):
      ![Depth and width results](images/Wide_Residual_Networks__depth_and_width.png?raw=true "Depth and width results")
    * They argue that their results show no evidence that would support the common theory that thin and deep networks somehow regularized better than wide and shallow(er) networks.
  * *Dropout*:
    * They use dropout with p=0.3 (CIFAR) and p=0.4 (SVHN).
    * On CIFAR-10 dropout doesn't seem to consistently improve test error.
    * On CIFAR-100 and SVHN dropout seems to lead to improvements that are either small (wide and shallower net, i.e. depth=28, width multiplier=10) or significant (ResNet-50).
      ![Dropout](images/Wide_Residual_Networks__dropout.png?raw=true "Dropout")
    * They also observed oscillations in error (both train and test) during the training. Adding dropout decreased these oscillations.
  * *Computational efficiency*:
    * Applying few big convolutions is much more efficient on GPUs than applying many small ones sequentially.
    * Their network with the best test error is 1.6 times faster than ResNet-1001, despite having about 3 times more parameters.


