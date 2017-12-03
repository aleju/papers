# Paper

* **Title**: Critical Learning Periods in Deep Neural Networks
* **Authors**: Alessandro Achille, Matteo Rovere, Stefano Soatto
* **Link**: https://arxiv.org/abs/1711.08856
* **Tags**: Neural Network
* **Year**: 2017

# Summary

* What
  * They find that artificial neural networks show critical periods, similar to biological neural networks.
  * Critical periods in biological neural networks are phases in the network's development
    during which a malformed sensory input
    can irreversibly harm the capabilities of the network.
    * E. g. a disease causing blurry vision for some time during early development
      can permanently harm the ability to interpret visual stimuli.
      The same disease *very* early or later in life will cause no permanent damage (after being healing).

* How
  * Blur
    * They train networks in CIFAR10 and blur all input images by down- and then upscaling.
    * They vary when the blurring is removed.
    * They always train for 300 epochs more after the blurring was removed.
      So the network always sees the unaltered input images for at least the same amount of epochs.
  * Vertical flipping
    * Same as blurring, but they vertically flip the image.
    * This is expected to keep low and mid level statistics the same. Only the final layers have to change.
    * This is expected to be easy to adapt to, even if the flipping is removed fairly late into the training.
  * Label permutation
    * They randomly permute the class labels and remove the effect after some epochs.
    * This is expected to only affect the last layer and hence should have similar effects to vertical flipping.
  * Sensory deprivation
    * They test the effect of sensory deprivation on neural nets.
    * They make the input of the networks uninformative by replacing it with gaussian noise.
    * This is assumed to have less effect than adding blur, because the network does not learn significantly wrong statistics (due to the input being uninformative).
  * Mutual Information Noise
    * They add random gaussian noise to each layer's output (or to the weights - not really clear).
    * They allow the network to learn the variance of that noise.
    * They add a regularization based on mutual information.
      This adds a "cost proportional to the quantity of mutual information I(w;D) that the weights retain about the training data D after the learning process" (?).
    * So the network can retain more information, but has to pay for that.
    * It is expected to set the variance to low values for layers which are critical for the predictions.

* Results
  * Blur
    * Removing the blur early leads to only a small loss in final accuracy.
    * Removing the blur too late leads to a large and permanent loss in final accuracy.
    * The decline in accuracy is not linear with respect to when the blur is removed.
    * The effect is similar to biological neural nets.
    * ![blur](images/Critical_Learning_Periods_in_Deep_Neural_Networks/blur.jpg?raw=true "blur")
    * Making the network deeper does not help, but instead worsens the effect.
    * Fixing the learning rate doesn't help either.
    * ![blur depth](images/Critical_Learning_Periods_in_Deep_Neural_Networks/blur_depth.jpg?raw=true "blur depth")
    * This is basically like starting to let the network learn once the blur is removed,
      but using a weirdly bad initialization.
      (Weird in the sense that it starts with great accuracy, but is barely able to improve.)
  * Vertical flipping
    * As expected, adding vertical flips does not significantly affect long term accuracy.
  * Label permutation
    * Same as for vertical flipping, only minor effect.
  * Sensory deprivation
    * This has worse effects than vertical flipping / label permutation.
    * Overall less decrease in accuracy than with blur.
    * The effect is more linear with respect to the epoch (remove early: hardly any decline in accuracy, remove after half of training: medium decline, remove late: strong decline).
  * Mutual Information Noise
    * Without deficit, the network will put most weight (least amount of noise) on the middle layers (3-5 of 7).
    * With deficit, it will put more weight on the last layers and is only able to partially reconfigure if the deficit is removed early enough.
    * ![information per layer](images/Critical_Learning_Periods_in_Deep_Neural_Networks/information_per_layer.jpg?raw=true "information per layer")

