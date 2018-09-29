# Paper

* **Title**: Towards Realistic Predictors
* **Authors**: Pei Wang, Nuno Vasconcelos
* **Link**: [http://openaccess.thecvf.com/content_ECCV_2018/papers/Pei_Wang_Towards_Realistic_Predictors_ECCV_2018_paper.pdf](http://openaccess.thecvf.com/content_ECCV_2018/papers/Pei_Wang_Towards_Realistic_Predictors_ECCV_2018_paper.pdf)
* **Tags**: Neural Network, Uncertainty, ECCV 2018
* **Year**: 2018

# Summary

* What
  * They propose a method for classifiers to estimate how hard/difficult an example is (for the specific classifier).
  * They call that "realistic predictors" (in the sense of "being realistic with respect to one's own abilities").
  * Their method allows them to
    1. focus training on hard examples,
    2. reject too difficult inputs,
    3. based on point (2) guarantee average accuracies by rejecting inputs with certain thresholds of diffculty-estimates.
       (This is useful e.g. for some safety critical systems, where no decision might be better than a wrong decision.
        It can also be used for combinations of models and humans, where a model would automate easy tasks and a few humans deal with the remaining hard tasks.)

* How
  * Architecture
    * They have a given classifier `F`.
    * They need per example `i` a hardness estimate `s_i` denoting how difficult that input is for `F`.
    * They predict that value with `HP-Net` (hardness predictor), a network completely separate from `F`.
    * The estimated hardness values `s_i` influence the training of `F` for hard negative mining.
    * The predictions of `F` are used during the training of `HP-Net`.
    * Visualization:
      * ![architecture](images/Towards_Realistic_Predictors/architecture.jpg?raw=true "architecture")
  * Loss
    * For `F` (classifier):
      * `F` predicts for multiple classes a probability value (after softmax).
      * They use a crossentropy loss weighted by the hardness:
        * ![loss F](images/Towards_Realistic_Predictors/loss_f.jpg?raw=true "loss F")
        * Here, `s_i` is the hardness if the i-th example and `p_i^c` is the probability predicted by `F` for the correct class of the i-th example.
        * This increases the loss for hard examples (high `s_i` value) and decreases it for easy examples (low `s_i`).
    * For `HP-Net` (hardness predictor):
      * `HP-Net` predicts the inverse of the probability that `F` is going to predict for the correct class.
        So if `F` predicts for the correct class a high probability, `HP-Net` is trained to predict a low hardness value and vice versa.
      * They use a binary crossentropy loss between `p_i^c` (`F`'s prediction for the correct class) and `s_i` (hardness):
        * ![loss HP-Net](images/Towards_Realistic_Predictors/loss_hp_net.jpg?raw=true "loss HP-Net")
  * Training Schedule and Refinement
    * The full training schedules happens in the following steps.
      1. Train `F` and `HP-Net` jointly on a training set.
         Do that training iteratively (one batch for `F`, then one batch for `HP-Net`). 
         Otherwise the networks would not converge for them.
      2. Use `HP-Net` to eliminate hard examples from the dataset. (E.g. the hardest 5%.)
      3. Train a new `F` on reduced dataset, result is `F'` (with higher accuracy for these easier examples).
         Reuse `HP-Net` from (1) during this training, keep its weights fixed.
      4. Output pair `(F', HP-Net)`.
   * Then, at test time examples above a certain hardness threshold are rejected (similar to step 2) and `F'` is applied to the remaining examples.
   * Visualization:
     * ![schedule](images/Towards_Realistic_Predictors/schedule.jpg?raw=true "schedule")

* Results
  * They test on MNIST, MIT67 and ImageNet.
  * They test for both `F` and `HP-Net` the networks VGG, ResNet, kerasNet and LeNet5. (They slightly modify the heads for `HP-Net`.)
  * They test with and without weight sharing between `F` and `HP-Net`.
  * They observe that hardness scores predicted by `HP-Net` start with high values and then shift towards lower values during training. (As expected.)
  * They achieve significantly worse accuracies if using weight sharing between `F` and `HP-Net`. This indicates that the two networks solve fairly different tasks.
  * They achieve the highest accuracies when using the same network architectures for `F` and `HP-Net` (e.g. both ResNet). This indicates that the `HP-Net` has to operate in a similar way to `F` in order to predict its predictions.
  * On ImageNet
    * They get almost 1 percentage point higher accuracy, even when not rejecting any examples. This is likely due to the indirect hard negative mining in the loss of `F`.
    * Rejecting the 5% most difficult examples results in 0.7 points higher accuracy (on the remaining ones). At 10% rejection rate they get 1.3 higher accuracy.
    * One can reject examples based on the hardness score or based on the highest class probability predicted by `F`.
      Doing it based on the hardness score performs marginally better (+0.1 points) for high rejection rates and decently better (+0.7 points) for a rejection rate of 5%.
    * Refining on the reduced dataset (from `F` to `F'`) gives around 0.1 points higher accuracy.
    * To reach with VGG the accuracy of ResNet at 2% rejection rate, one has to set VGG's rejection rate to 10%.

