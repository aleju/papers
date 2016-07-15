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
    * Target of this step: Get rid of variations within the face images, so that every face seems to look straight into the camera ("frontalized").
    * 2D alignment
      * They search for landmarks (fiducial points) on the face.
        * They use SVRs (features: LBPs) for that.
        * After every application of the SVR, the localized landmarks are used to transform/normalize the face. Then the SVR is applied again. By doing this, the locations of the landmarks are gradually refined.
      * They use the detected landmarks to normalize the face images (via scaling, rotation and translation).
    * 3D alignment
      * The 2D alignment allows to normalize variations within the 2D-plane, not out-of-plane variations (e.g. seeing that face from its left/right side). To normalize out-of-plane variations they need a 3D transformation.
      * They detect an additional 67 landmarks on the faces (again via SVRs).
      * They construct a human face mesh from a dataset (USF Human-ID).
      * They map the 67 landmarks to that mesh.
      * They then use some more complicated steps to recover the frontalized face image.
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
    * They normalize "the features" (probably the 4096 fully connected layer). Each component is divided by its maximum value across a training set. Additionally, the whole vector is L2-normalized. The goal of this step is to make the network less sensitive to illumination changes.
    * The whole network has about 120 million parameters.
    * Visualization of the architecture:
      * ![Architecture](images/DeepFace__architecture.jpg?raw=true "Architecture")
  * *Training*
    * The network receives images, each showing a face, and is trained to classify the identity of the face (e.g. gets image of Obama, has to return "that's Obama").
    * They use cross-entropy as their loss.
  * *Face verification*
    * In order to tell whether two images of faces show the same person they try three different methods.
    * Each of these relies on the vector extracted by the first fully connected layer in the network (4096d).
    * Let these vectors be `f1` (image 1) and `f2` (image 2). The methods are then:
      1. Inner product between `f1` and `f2`. The classification (same person/not same person) is then done by a simple threshold.
      2. Weighted X^2 (chi-squared) distance. Equation, per vector component i: `weight_i (f1[i] - f2[i])^2 / (f1[i] + f2[i])`. The vector is then fed into an SVM.
      3. Siamese network. Means here simply that the absolute distance between `f1` and `f2` is calculated (`|f1-f2|`), each component is weighted by a learned weight and then the sum of the components is calculated. If the result is above a threshold, the faces are considered to show the same person.

* Results
  * They train their network on the Social Face Classification (SFC) dataset. That seems to be a Facebook-internal dataset (i.e. not public) with 4.4 million faces of 4k people.
  * When applied to the LFW dataset:
    * Face recognition ("which person is shown in the image") (apparently they retrained the whole model on LFW for this task?):
      * Simple SVM with LBP (i.e. not their network): 91.4% mean accuracy.
      * Their model, with frontalization, with 2d alignment: ??? no value.
      * Their model, no frontalization (only 2d alignment): 94.3% mean accuracy.
      * Their model, no frontalization, no 2d alignment: 87.9% mean accuracy.
    * Face verification (two images -> same/not same person) (apparently also trained on LFW? unclear):
      * Method 1 (inner product + threshold): 95.92% mean accuracy.
      * Method 2 (X^2 vector + SVM): 97.00% mean accurracy.
      * Method 3 (siamese): Apparently 96.17% accuracy alone, and 97.25% when used in an ensemble with other methods (under special training schedule using SFC dataset).
  * When applied to the YTF dataset (YouTube video frames):
    * 92.5% accuracy via X^2-method.
