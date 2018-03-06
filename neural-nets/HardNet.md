# Paper

* **Title**: Working hard to know your neighbor’s margins: Local descriptor learning loss
* **Authors**: Anastasiya Mishchuk, Dmytro Mishkin, Filip Radenovic, Jiri Matas
* **Link**: https://arxiv.org/abs/1705.10872
* **Tags**: Neural Network, Loss functions
* **Year**: 2017

# Summary

* **What:**
  * HardNet model that which improves state-of-the-art in wide baseline stereo, patch matching, verification and retrieval and in image retrieval.
  * They introduced a new triplet-like loss function with built-in hard-negative mining.
![Mining Procedure](images/HardNet/Triplet_mining.png?raw=true "Mining Procedure")

* **How:**
  * HardNet Triplet loss is a regular Triplet-Loss = MAX(0, `alpha` + `distances_to_positives` - `distances_to_negatives`), where:
    * `alpha` (sometimes called as margin) is a hyper-parameter
    * `distance_to_positives` = distances (in the current article L2 is taken)
    * `distance_to_negative` = distances to the hardest negatives for each anchors in a batch.
  * As input HardNet operates with N x 2 images (N anchor/query images and N corresponding to them positives)
  * Mining algorithm:
        1. Compute distance matrix `D` between N anchors and N positives.
        2. `distances_to_positives` = trace of distance matrix (diagonal elements)
        3. For each row minimal non-diagonal element is taken as a distance to the hardest negatives (closest to anchor). From these chosen values `distances_to_negatives` are obtained.
    All this can be rewritten as:
        Loss = MAX(0, `alpha` + Trace(`D`) + row_wise_min(`D` + `I` * `inf`)), where I is a identity matrix.
   * Architecture:
     ![Architecture](images/HardNet/Architecture.png?raw=true "Architecture")

* **Notes:**
  * The described mining procedure highly relies on a fact that all N anchors would should to N different classes.
    And from my personal point of view requires minor modification to handle such corner case.
  * The given loss/mining procedure is fast, but in contrast to other mining strategies doesn't provide hardest positive (furthest from anchor).

* **Results:**
  * Wide baseline stereo example:\
  ![Example](images/HardNet/Wide_baseline_stereo_examples.png?raw=true "Wide baseline stereo example")
  * The bigger batch size, the better:\
  ![BatchSizeInfluence](images/HardNet/BatchSize_influence.png?raw=true "Batch Size Influence")
  * PhotoTour Patch Verification Results\
  ![PhotoTour Patch Verification](images/HardNet/PhotoTour_results.png?raw=true "PhotoTour Patch Verification")
  * Oxford 5k, Paris 6k Patch Verification Results\
  ![Oxford 5k, Paris 6k Patch Verification Results](images/HardNet/Oxford5k,Paris6k_results.png?raw=true "Oxford 5k, Paris 6k Patch Verification Results")


