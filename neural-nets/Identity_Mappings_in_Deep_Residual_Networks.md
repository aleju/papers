# Paper

* **Title**: Identity Mappings in Deep Residual Networks
* **Authors**: Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun
* **Link**: http://arxiv.org/abs/1603.05027v2
* **Tags**: Neural Network, residual
* **Year**: 2016

# Summary

* What
  * The authors reevaluate the original residual design of neural networks.
  * They compare various architectures of residual units and actually find one that works quite a bit better.

* How
  * The new variation starts the transformation branch of each residual unit with BN and a ReLU.
  * It removes BN and ReLU after the last convolution.
  * As a result, the information from previous layers can flow completely unaltered through the shortcut branch of each residual unit.
  * The image below shows some variations (of the position of BN and ReLU) that they tested. The new and better design is on the right:
    ![BN and ReLU positions](images/Identity_Mappings_in_Deep_Residual_Networks__activations.png?raw=true "BN and ReLU positions")
  * They also tried various alternative designs for the shortcut connections. However, all of these designs performed worse than the original one. Only one (d) came close under certain conditions. Therefore, the recommendation is to stick with the old/original design.
    ![Shortcut designs](images/Identity_Mappings_in_Deep_Residual_Networks__shortcuts.png?raw=true "Shortcut designs")

* Results
  * Significantly faster training for very deep residual networks (1001 layers).
  * Better regularization due to the placement of BN.
  * CIFAR-10 and CIFAR-100 results, old vs. new design:
    ![Old vs new results](images/Identity_Mappings_in_Deep_Residual_Networks__old_vs_new.png?raw=true "Old vs new results")
