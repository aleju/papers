# Paper

- **Title**: Generic Model-Agnostic Convolutional Neural Network for Single Image Dehazing
- **Authors**: Zheng Liu, Botao Xiao, Muhammad Alrabeiah, Keyan Wang, Jun Chen
- **Link**: https://arxiv.org/abs/1810.02862
- **Tags**: Neural Network, Image Dehazing
- **Year**: 2015

# See Also
- **[Code (Implementation in TensorFlow v2+)](https://github.com/sanchitvj/Image-Dehazing-using-GMAN-net), [Explaination](tinyurl.com/gman-dehaze-net)** 

# Summary
- What
    - Haze and smog are among the most common environmental factors impacting image quality and, therefore, image analysis. This paper proposes an end-to-end generative method for image dehazing.
    - It is based on designing a fully convolutional neural network to recognize haze structures in input images and restore clear, haze-free images.
    - Functionally addressing, this architecture is composed of an end-to-end generative method which uses an encoder-decoder approach to solve the dehazing problem.
    - Input is a synthetically hazed image from [SOTS Outdoor](https://www.kaggle.com/wwwwwee/dehaze) dataset. Output is a dehazed image.
    
- How
    - *Architecture*
      - The first two layers which input image encounters are convolution blocks with 64 channels. Following them are 2 downsampling blocks(encoder) with stride 2. The encoded image is then fed to a layer build with 4 residual blocks.    
      - Each block contained a shortcut connection(same as ResNets). This residual layer helps to understand the haze structures. After these comes the upsampling or deconvolutional (decoder) layer which reconstructs the output of residual layers.  
      - The last two layers are convolutional blocks that transform the upsampled feature map into a 3 channel RGB image which finally added to the input image(global residual layer) and then ReLU to give dehazed output.  
      - This global residual layer helps in capturing the boundary details of objects with different depths in the scene.
      - The encoder part of architecture helps to decrease the dimension of the image and then fed the downsampled image to the residual layer to extract the image features and the decoder part is expected to learn and regenerate the missing data of the haze-free image.
      ![image](https://miro.medium.com/max/875/1*nE7o6HsOrdUUff3_XJoXyQ.png)
      ![img1](https://miro.medium.com/max/258/1*WPqNf1ykPrSk4QnQMAvKQg.png)
      
     - *Encoder-Decoder*
        - This architecture makes it possible to train a deep network and decrease the dimension of data. Since haze could be thought of as a form of noise, the encoder output is downsampled and fed to the residual layer to extract important features.
        - The network squeezes out the features of the original image and discards of the noise information. The decoder part is expected to learn and regenerate the missing data of the haze-free image, conforming the statistical distribution of the input information during the decoding period.
        
     - *Training*
        - All layers in GMAN have 64 filters (kernels), except for the down-sampling ones which have 128 filters, with spatial size of 3*3. The network requires an input with size 224*224, so every image in the training dataset is randomly cropped in order to fit the input size3.
        - The batch size is set to 35 to balance the training speed and the memory consumption on the GPU. For accelerated training, the Adam optimizer is used with the following settings: the initial learning rate of 0.001, 1 = 0:9, and 2 = 0:999.
        - The network and its training process have been implemented using TensorFlow software framework and carried out on an NVIDIA Titan Xp GPU. After 20 epochs of training, the loss function drops to a value of 0.0004, which is considered a good stopping point.
        
- Results
  - 
