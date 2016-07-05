# Paper

* **Title**: DeepFace: Closing the Gap to Human-Level Performance in Face Verification
* **Authors**: Yaniv Taigman, Ming Yang, Marc’Aurelio Ranzato, Lior Wolf
* **Link**: https://www.cs.toronto.edu/~ranzato/publications/taigman_cvpr14.pdf
* **Tags**: Neural Network, face
* **Year**: 2014

# Summary

* What
  * They describe a CNN architecture that can be used to identify a person given an image of their face.

* How
  * The expected input is the image of a face (i.e. it does not search for faces in images, the faces already have to be extracted by a different method).
  * *Face alignment / Frontalization*
    * They search for landmarks (fiducial points) on the face.
    * They use these points to reconstruct a 3d model of the face (including e.g. the rotation).
    * They then transform that 3d model to look straight into the camera.
    * They then project the 3d model onto a 2d plane.
    * The result is an approximation of the face as if it looked straight into the camera ("frontalized").
    * This preprocessing is used to decrease the complexity of the function that the neural net has to learn (i.e. to make the learning task easier).
  * *CNN architecture*
    * The CNN receives the frontalized face images (152x152, RGB).
    * It then applies the following steps:
      * Convolution, 32 filters, 11x11, ReLU (-> 32x142x142, CxHxW)
      * Max pooling over 3x3, stride 2 (-> 32x71x71)
      * Convolution, 16 filters, 9x9, ReLU (-> 16x63x63)
      * Local Convolution, 16 filters, 9x9, ReLU (-> 16x55x55)
      * Local Convolution, 16 filters, 7x7, ReLU (-> 16x25x25)
      * Local Convolution, 16 filters, 5x5, ReLU (-> 16x21x21)
      * Fully Connected, 4096, ReLU
      * Fully Connected, 4030, Softmax
    * Local Convolutions use a different set of learned weights at every "pixel" (while a normal convolution uses the same set of weights at all locations).
    * They can afford to use local convolutions because of their frontalization, which roughly forces specific landmarks to be at specific locations.
    * They use dropout (apparently only after the first fully connected layer).
    * They use cross-entropy as their loss.
  * 

* Results

![Examples](images/Accurate_Image_Super-Resolution__examples.png?raw=true "Examples")

*Super-resolution quality of their model (top, bottom is a competing model).*
