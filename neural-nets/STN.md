# Paper

* **Title**: Spatial Transformer Network
* **Authors**: Max Jaderberg, Karen Simonyan, Andrew Zisserman, Koray Kavukcuoglu
* **Link**: https://arxiv.org/abs/1506.02025
* **Tags**: Neural Network, Attention
* **Year**: 2015

# Summary

* **What:**
  * They introduced a new learnable module, the Spatial Transformer, which explicitly allows the spatial manipulation of data within the network.
  ![STN](images/STN/stn.png?raw=true "STN")

* **How:**
   * `Spatial Transformer` allows the spatial manipulation of the data (any feature map or particularly input image). This differentiable module can be inserted into any CNN, giving neural networks the ability to actively spatially transform feature maps, conditional on the feature map itself.
   * The action of the spatial transformer is conditioned on individual data samples, with the appropriate behavior learned during training for the task in question.
   * No additional supervision or modification of the optimization process is required.
   * Spatial manipulation consists of cropping, translation, rotation, scale, and skew.
   ![Example](images/STN/stn_example2.png?raw=true "Example") ![Example2](images/STN/stn_example.png?raw=true "Example2")
   * STN structure:
        1. `Localization net`: predicts parameters of the transform `theta`. For 2d case, it's 2 x 3 matrix. For 3d case, it's 3 x 4 matrix.
        2. `Grid generator`: Uses predictions of `Localization net` to create a sampling grid, which is a set of points where the input map should be sampled to produce the transformed output.
        3. `Sampler`: Produces the output map sampled from the input feature map at the predicted grid points.
    
* **Notes**:
    * Localization net can predict several transformations(`thetas`) for subsequent transformation applied to the input image(feature map).  
        * The final regression layer should be initialized to regress the identity transform (zero weights, identity transform bias).
    * *Grid generator* and *Transforms*: 
        * The transformation can have any parameterized form, provided that it is differentiable with respect to the parameters
        * The most popular is just a 2d affine transform: \
         ![2dAffine](images/STN/stn_2d_spatial_transform.png?raw=true "2D Affine Transform") 
         or particularly an attention mechanism: \
          ![Attention](images/STN/stn_attention_thetas.png?raw=true "Attention")  
        * The source/target transformation and sampling is equivalent to the standard texture mapping and coordinates used in graphics.
    * *Sampler*:
        * **The key why STN works.** They introduced a (sub-)differentiable sampling mechanism that allows loss gradients to flow back not only to the "input" feature map, but also to the sampling grid coordinates, and therefore back to the transformation parameters θ and *Localisation Net*.
              
     
* **Results:**
    * **Street View House Numbers multi-digit recognition**:
      ![SVHN Results](images/STN/stn_svhn_results.png?raw=true   "SVHN Results")
    * **Distored MNIST**:
      ![Distorted MNIST Results](images/STN/stn_distored_mnist_results.png?raw=true "Distorted MNIST")
    * **CUB-200-2011 birds dataset**:
      ![Birds Classification Results](images/STN/stn_birds_results.png?raw=true "Birds Classification Results")
    * **MNIST addition**:
      ![MNIST addition Results](images/STN/stn_mnist_addition_results.png?raw=true "MNIST addition Results")  
