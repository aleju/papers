# Paper

* **Title**: Instance Normalization: The Missing Ingredient for Fast Stylization
* **Authors**: Dmitry Ulyanov, Andrea Vedaldi, Victor Lempitsky
* **Link**: https://arxiv.org/abs/1607.08022
* **Tags**: Neural Network, style transfer, normalization
* **Year**: 2016

# Summary

* What
  * Style transfer between images works - in its original form - by iteratively making changes to a content image, so that its style matches more and more the style of a chosen style image.
  * That iterative process is very slow.
  * Alternatively, one can train a single feed-forward generator network to apply a style in one forward pass. The network is trained on a dataset of input images and their stylized versions (stylized versions can be generated using the iterative approach).
  * So far, these generator networks were much faster than the iterative approach, but their quality was lower.
  * They describe a simple change to these generator networks to increase the image quality (up to the same level as the iterative approach).

* How
  * In the generator networks, they simply replace all batch normalization layers with instance normalization layers.
  * Batch normalization normalizes using the information from the whole batch, while instance normalization normalizes each feature map on its own.
  * Equations
    * Let `H` = Height, `W` = Width, `T` = Batch size
    * Batch Normalization:
      * ![Batch Normalization Equations](images/Instance_Normalization_The_Missing_Ingredient_for_Fast_Stylization__batch_normalization.jpg?raw=true "Batch Normalization Equations")
    * Instance Normalization
      * ![Instance Normalization Equations](images/Instance_Normalization_The_Missing_Ingredient_for_Fast_Stylization__instance_normalization.jpg?raw=true "Instance Normalization Equations")
  * They apply instance normalization at test time too (identically).

* Results
  * Same image quality as iterative approach (at a fraction of the runtime).
  * One content image with two different styles using their approach:
    * ![Example](images/Instance_Normalization_The_Missing_Ingredient_for_Fast_Stylization__example.jpg?raw=true "Example")
