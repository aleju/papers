# Paper

* **Title**: Snapshot Ensembles: Train 1, get M for free
* **Authors**: Gao Huang, Yixuan Li, Geoff Pleiss, Zhuang Liu, John E. Hopcroft, Kilian Q. Weinberger
* **Link**: https://arxiv.org/abs/1704.00109
* **Tags**: Neural Network, ensemble
* **Year**: 2017

# Summary

* What
  * They suggest a method to generate ensembles of models requiring less total training time.
  * The method is based on "saving" intermediary versions of models.

* How
  * They save every M epochs an intermediary version of the model (i.e. they save the parameters).
    Then they combine the last n models to an ensemble.
  * To make each version more dissimilar from the other ones, they cycle the learning rate using a cosine function.
    That means that they increase the learning rate significantly, then keep it at the high level for a short time,
    then decrease it fast, then keep it at a low level for a short time.
  * Formula for the (cycled) learning rate:
    * ![formula](images/Snapshot_Ensembles/formula.jpg?raw=true "formula")
    * `alpha_0` is the initial learning rate,
    * `T` the total number of training iterations
    * `M` the number of training iterations per cycle (after each cycle, the learning rate is increased)
  * Visualization of the cycled learning rate:
    * ![cycles](images/Snapshot_Ensembles/cycles.jpg?raw=true "cycles")
    * (Note that after the third cycle the model is already at nearly optimal accuracy, despite having suffered two times from
       increasing the learning rate. With a better learning rate schedule it might have been able to reach the optimum in
       2-3 cycles. So it is kinda unhonest to say that this ensembling method adds no training it, it just probably adds
       less than "normal" ensembling from scratch.)
    * (Also note here that the blue learning rate schedule that they are comparing against is probably far away from being optimal.)
  * They argue that cycling the learning rate is also useful to "jump" between local minima.
    I.e. the model reaches a local minima, then escapes it using a high learning rate,
    then descends into a new local minima using the lowered learning rate.
    Then the ensemble would consist of models in different local minima.
    * (Note here though that the current state of science is that there aren't really local minima in deep NNs,
      only saddle points.)
    * (Also note that this means that the ensembled models probably are often fairly similar.
       A proper ensemble consisting only of models trained from scratch might perform better.)

* Results
  * Using roughly the 3 last snapshots for the ensemble seems to be the best compromise (at `alpha_0=0.1`).
  * Using too many snaphots can worsen the results.
  * Using `alpha_0=0.2` seems to be a better choice than `alpha_0=0.1`.
    They argue that the high learning rate between cycles leads to more diverse local minima.
  * Only running one learning rate cycle and collecting the ensemble models from that leads to worse results (as opposed to running multiple cycles).
    The following visualization shows the effect of using a single cycle vs. multiple (in relations to the training iterations).
    * ![single cycle](images/Snapshot_Ensembles/single_cycle.jpg?raw=true "single cycle")
  * They perform overall better than models without ensembling.
    * ![results](images/Snapshot_Ensembles/results.jpg?raw=true "results")
  * True ensembles reach quite a bit better accuracy still.
    * ![true ensembles](images/Snapshot_Ensembles/true_ensembles.jpg?raw=true "true ensembles")
  * Running models with interpolated snaphots (e.g. set each weight to 30% of snapshot 1 and 70% of snaphot 5),
    the test accuracy is improved if the snapshots are close to each other (e.g. snaphot 4 and 5).
    This indicates that the parameters change more and more with each cycle.

