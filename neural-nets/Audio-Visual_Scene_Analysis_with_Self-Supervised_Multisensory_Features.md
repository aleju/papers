# Paper

* **Title**: Audio-Visual Scene Analysis with Self-Supervised Multisensory Features
* **Authors**: Andrew Owens, Alexei A. Efros
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Andrew_Owens_Audio-Visual_Scene_Analysis_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Andrew_Owens_Audio-Visual_Scene_Analysis_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Audio, Sound Source Localization, Action Recognition, Sound Source Separation, Self-Supervised, ECCV 2018
* **Year**: 2018

# See also

[Video](https://www.youtube.com/watch?v=rwCIRu_hAJ8)

# Summary

* What
  * They propose a method to pretrain jointly on video and corresponding audio data without labels and then use the results to:
    * perform sound source logalicization (estimate what in an image causes the current sound),
    * perform audio-visual action recognition,
    * perform audio-separation (e.g. remove speech from person not visible in an image, remove speech from a specific visible person).

* How
  * Basic method
    * They train their network to differentiate between real frames (4.2 seconds) and their corresponding audio data on the one hand
      and real frames but misaligned audio data on the other hand.
    * Misaligned here means that the audio is shifted by a few seconds (by 2.0 to 5.8 seconds to be precise).
    * To solve this task, the network has to learn to understand the scene in order to spot small misalignments.
    * The network can learn this task with self-supervision, i.e. without annotations.
  * Architecture
    * Their network starts with two branches.
    * One for frames from videos.
    * One for audio data.
    * Both branches are then merged and finally end in a fully convolutional layer.
    * Visualization:
      * ![architecture](images/Audio-Visual_Scene_Analysis_with_Self-Supervised_Multisensory_Features/architecture.jpg?raw=true "architecture")
  * Sound Source Localization
    * They train their model as described.
    * The features maps of the final convolutional layer then contain information about the spatial location of the sound source.
    * They convert these feature maps to a heatmap visualization by first weighting each map (based on its weight to produce the final binary output),
      then summing the maps and lastly normalizing them to e.g. 0 to 1.0.
  * Action Recognition
    * They first pretrain using their described method, then fine-tune on the UCF-101 dataset (using 2.56 second videos with augmentation and small audio-shifts down to one frame).
  * On/off-screen audio-visual source separation
    * Their goal is to separate sound from (a) sources visible in the video and (b) sources not visible in the video.
      * Sources in (b) can be outside of the camera view, but can also be sources intentionally hidden, e.g. by setting the image area of a known source in all frames to zero.
      * They focus here in speech by people (i.e. the model has to learn to separate between speeches by people visible in the video and those not visible).
    * They first pretrain a model as described above.
    * Then they create a new model:
      * It has one subnetwork for video+audio processing (can reuse the weights of the pretrained network).
      * It has a second subnetwork to process the audio spectrogram in a U-Net form.
      * The features of the first subnetwork get merged into the second subnetwork at various resolutions.
      * The output are two spectrograms: one for the on-screen audio, one for the off-screen audio.
      * Visualization:
        * ![architecture sound separation](images/Audio-Visual_Scene_Analysis_with_Self-Supervised_Multisensory_Features/architecture_sound_separation.jpg?raw=true "architecture sound separation")
    * They train this by pairing video sequences from VoxCeleb with their audio (on-screen) and other audio from a random clip (off-screen).
    * They use a loss with two L1-based regression losses:
      * ![loss sound separation](images/Audio-Visual_Scene_Analysis_with_Self-Supervised_Multisensory_Features/loss_sound_separation.jpg?raw=true "loss sound separation")
      * where `x_F` is the on-screen (foreground) audio, `f_F(.)` its prediction and analogously `x_B` and `f_B(.)` the background (off-screen) audio and its prediction.
      * One can make this loss permutation invariant by using `min(L(x_M, I), L(I, x_M))`.

* Results
  * Sound Source Localization
    * They train on 750k videos from the AudioSet dataset.
    * They reached 59.9% accuracy during pretraining (i.e. when discriminating between non-shifted and shifted audio). Humans reached around 66.6% (under easier test conditions, e.g. stronger audio shifts).
    * Qualitative results, showing via CAM method the test examples with strongest activation maps:
      * ![sound sources strong](images/Audio-Visual_Scene_Analysis_with_Self-Supervised_Multisensory_Features/sound_sources_strong.jpg?raw=true "sound sources strong")
      * Note that AudioSet did not only include human speech. The model just seemed to react strongly to that.
    * Same as before, but test examples with the weakest activation maps:
      * ![sound sources weak](images/Audio-Visual_Scene_Analysis_with_Self-Supervised_Multisensory_Features/sound_sources_weak.jpg?raw=true "sound sources weak")
    * Strongest activation maps, separated for three example classes (trained on Kinetics):
      * ![sound sources strong classwise](images/Audio-Visual_Scene_Analysis_with_Self-Supervised_Multisensory_Features/sound_sources_strong_classwise.jpg?raw=true "sound sources strong classwise")
  * Action Recognition
    * They used the UCF-101 dataset, as mentioned above.
    * They achieve significantly higher scores than other self-supervised methods.
    * They are still quite a bit behind the best method (82.1% vs 94.5%), though that one is a supervised method and was pretrained on two large datasets (ImageNet + Kinetics).
    * They test here pairing videos with audio from random other videos (instead of shifted audio from the same video).
      This reduced performance from 84.2% to 78.7%, likely because it is a too easy task that often does not require precise understanding of e.g. the motions in the video.
    * Setting the audio branch's activation after pretraining to zero and then training on UCF-101 resulted in an accuracy drop of 5 percentage points.
    * Using no self-supervised pretraining resulted in an accuracy drop of 14 percentage points.
    * Using spectrograms as audio input performed comparable to raw wave inputs.
  * On/off-screen audio-visual source separation
    * They trained on the VoxCeleb dataset, as described above.
      * They made sure that the speaker identities were disjoint between train and test sets.
      * They sampled 2.1s clips from each 5s video.
    * Results are shown in the video (see at top of this file).
    * Training the model only on a single frame instead of the full video worsened the loss from 11.4 to 14.8.
      * This drop was especially pronounced in videos with two persons of the same gender, where lip motion is an important clue.
    * Using the state-of-the-art I3D net pretrained on Kinect as audio-visual feature inputs (instead of their pretrained net) worsened the loss from 11.4 to 12.3.
      * They suspect that this is because action recognition can work decently with just single-frame analysis, while their pretraining (and this source seperation task) require in-depth motion analysis.

