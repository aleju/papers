# Paper

* **Title**: Artistic style transfer for videos
* **Authors**: Manuel Ruder, Alexey Dosovitskiy, Thomas Brox
* **Link**: http://arxiv.org/abs/1604.08610
* **Tags**: Neural Network, style transfer, video
* **Year**: 2016

# Summary

* What
  * The paper describes a method to transfer the style (e.g. choice of colors, structure of brush strokes) of an image to a whole video.
  * The method is designed so that the transfered style is consistent over many frames.
  * Examples for such consistency:
    * No flickering of style between frames. So the next frame has always roughly the same style in the same locations.
    * No artefacts at the boundaries of objects, even if they are moving.
    * If an area gets occluded and then unoccluded a few frames later, the style of that area is still the same as before the occlusion.

* How
  * Assume that we have a frame to stylize `x` and an image from which to extract the style `a`.
  * The basic process is the same as in the original [Artistic Style Transfer paper](A_Neural_Algorithm_for_Artistic_Style.md), they just add a bit on top of that.
  * They start with a gaussian noise image `x'` and change it gradually so that a loss function gets minimized.
  * The loss function has the following components:
    * Content loss *(old, same as in the Artistic Style Transfer paper)*
      * This loss makes sure that the content in the generated/stylized image still matches the content of the original image.
      * `x` and `x'` are fed forward through a pretrained network (VGG in their case).
      * Then the generated representations of the intermediate layers of the network are extracted/read.
      * One or more layers are picked and the difference between those layers for `x` and `x'` is measured via a MSE.
      * E.g. if we used only the representations of the layer conv5 then we would get something like `(conv5(x) - conv5(x'))^2` per example. (Where conv5() also executes all previous layers.)
    * Style loss *(old)*
      * This loss makes sure that the style of the generated/stylized image matches the style source `a`.
      * `x'` and `a` are fed forward through a pretrained network (VGG in their case).
      * Then the generated representations of the intermediate layers of the network are extracted/read.
      * One or more layers are picked and the Gram Matrices of those layers are calculated.
      * Then the difference between those matrices is measured via a MSE.
    * Temporal loss *(new)*
      * This loss enforces consistency in style between a pair of frames.
      * The main sources of inconsistency are boundaries of moving objects and areas that get unonccluded.
      * They use the optical flow to detect motion.
        * Applying an optical flow method to two frames (i, i+1) returns per pixel the movement of that pixel, i.e. if the pixel at (x=1, y=2) moved to (x=2, y=4) the optical flow at that pixel would be (u=1, v=2).
        * The optical flow can be split into the forward flow (here `fw`) and the backward flow (here `bw`). The forward flow is the flow from frame i to i+1 (as described in the previous point). The backward flow is the flow from frame i+1 to i (reverse direction in time).
      * Boundaries
        * At boundaries of objects the derivative of the flow is high, i.e. the flow "suddenly" changes significantly from one pixel to the other.
        * So to detect boundaries they use (per pixel) roughly the equation `gradient(u)^2 + gradient(v)^2 > length((u,v))`.
      * Occlusions and disocclusions
        * If a pixel does not get occluded/disoccluded between frames, the optical flow method should be able to correctly estimate the motion of that pixel between the frames. The forward and backward flows then should be roughly equal, just in opposing directions.
        * If a pixel does get occluded/disoccluded between frames, it will not be visible in one the two frames and therefore the optical flow method cannot reliably estimate the motion for that pixel. It is then expected that the forward and backward flow are unequal.
        * To measure that effect they roughly use (per pixel) a formula matching `length(fw + bw)^2 > length(fw)^2 + length(bw)^2`.
      * Mask `c`
        * They create a mask `c` with the size of the frame.
        * For every pixel they estimate whether the boundary-equation *or* the disocclusion-equation is true.
        * If either of them is true, they add a 0 to the mask, otherwise a 1. So the mask is 1 wherever there is *no* disocclusion or motion boundary.
      * Combination
        * The final temporal loss is the mean (over all pixels) of `c*(x-w)^2`.
          * `x` is the frame to stylize.
          * `w` is the previous *stylized* frame (frame i-1), warped according to the optical flow between frame i-1 and i.
          * `c` is the mask value at the pixel.
        * By using the difference `x-w` they ensure that the difference in styles between two frames is low.
        * By adding `c` they ensure the style-consistency only at pixels that probably should have a consistent style.
    * Long-term loss *(new)*
      * This loss enforces consistency in style between pairs of frames that are longer apart from each other.
      * It is a simple extension of the temporal (short-term) loss.
      * The temporal loss was computed for frames (i-1, i). The long-term loss is the sum of the temporal losses for the frame pairs {(i-4,i), (i-2,i), (i-1,i)}.
      * The `c` mask is recomputed for every pair and 1 if there are no boundaries/disocclusions detected, but only if there is not a 1 for the same pixel in a later mask. The additional condition is intended to associate pixels with their closest neighbours in time to minimize possible errors.
      * Note that the long-term loss can completely replace the temporal loss as the latter one is contained in the former one.
  * Multi-pass approach *(new)*
    * They had problems with contrast around the boundaries of the frames.
    * To combat that, they use a multi-pass method in which they seem to calculate the optical flow in multiple forward and backward passes? (Not very clear here what they do and why it would help.)
  * Initialization with previous frame *(new)*
    * Instead of starting at a gaussian noise image every time, they instead use the previous stylized frame.
    * That immediately leads to more similarity between the frames.

* Results
  * [Video](https://www.youtube.com/watch?v=vQk_Sfl7kSc&feature=youtu.be)
