# Paper

* **Title**: Swapout: Learning an ensemble of deep architectures
* **Authors**: Saurabh Singh, Derek Hoiem, David Forsyth
* **Link**: http://arxiv.org/abs/1605.06465v1
* **Tags**: Neural Network, dropout, residual, regularization
* **Year**: 2016

# Summary

* What
  * They describe a regularization method similar to dropout and stochastic depth.
  * The method could be viewed as a merge of the two techniques (dropout, stochastic depth).
  * The method seems to regularize better than any of the two alone.

* How
  * Let `x` be the input to a layer. That layer produces an output. The output can be:
    * Feed forward ("classic") network: `F(x)`.
    * Residual network: `x + F(x)`.
  * The standard dropout-like methods do the following:
    * Dropout in feed forward networks: Sometimes `0`, sometimes `F(x)`. Decided per unit.
    * Dropout in residual networks (rarely used): Sometimes `0`, sometimes `x + F(x)`. Decided per unit.
    * Stochastic depth (only in residual networks): Sometimes `x`, sometimes `x + F(x)`. Decided per *layer*.
    * Skip forward (only in residual networks): Sometimes `x`, sometimes `x + F(x)`. Decided per unit.
    * **Swapout** (any network): Sometimes `0`, sometimes `F(x)`, sometimes `x`, sometimes `x + F(x)`. Decided per unit.
  * Swapout can be represented using the formula `y = theta_1 * x + theta_2 * F(x)`.
    * `*` is the element-wise product.
    * `theta_1` and `theta_2` are tensors following bernoulli distributions, i.e. their values are all exactly `0` or exactly `1`.
    * Setting the values of `theta_1` and `theta_2` per unit in the right way leads to the values `0` (both 0), `x` (1, 0), `F(x)` (0, 1) or `x + F(x)` (1, 1).
  * Deterministic and Stochastic Inference
    * Ideally, when using a dropout-like technique you would like to get rid of its stochastic effects during prediction, so that you can predict values with exactly *one* forward pass through the network (instead of having to average over many passes).
    * For Swapout it can be mathematically shown that you can't calculate a deterministic version of it that performs equally to the stochastic one (averaging over many forward passes).
    * This is even more the case when using Batch Normalization in a network. (Actually also when not using Swapout, but instead Dropout + BN.)
    * So for best results you should use the stochastic method (averaging over many forward passes).

* Results
  * They compare various dropout-like methods, including Swapout, applied to residual networks. (On CIFAR-10 and CIFAR-100.)
  * General performance:
    * Results with Swapout are better than with the other methods.
    * According to their results, the ranking of methods is roughly: Swapout > Dropout > Stochastic Depth > Skip Forward > None.
  * Stochastic vs deterministic method:
    * The stochastic method of swapout (average over N forward passes) performs significantly better than the deterministic one.
    * Using about 15-30 forward passes seems to yield good results.
  * Optimal parameter choice:
    * Previously the Swapout-formula `y = theta_1 * x + theta_2 * F(x)` was mentioned.
    * `theta_1` and `theta_2` are generated via Bernoulli distributions which have parameters `p_1` and `p_2`.
    * If using fixed values for `p_1` and `p_2` throughout the network, it seems to be best to either set both of them to `0.5` or to set `p_1` to `>0.5` and `p_2` to `<0.5` (preference towards `y = x`).
    * It's best however to start both at `1.0` (always `y = x + F(x)`) and to then linearly decay them to both `0.5` towards the end of the network, i.e. to apply less noise to the early layers. (This is similar to the results in the Stochastic Depth paper.)
  * Thin vs. wide residual networks:
    * The standard residual networks that they compared to used a `(16, 32, 64)` pattern for their layers, i.e. they started with layers of each having 16 convolutional filters, followed by some layers with each having 32 filters, followed by some layers with 64 filters.
    * They tried instead a `(32, 64, 128)` pattern, i.e. they doubled the amount of filters.
    * Then they reduced the number of layers from 100 down to 20.
    * Their wider residual network performed significantly better than the deep and thin counterpart. However, their parameter count also increased by about `4` times.
    * Increasing the pattern again to `(64, 128, 256)` and increasing the number of layers from 20 to 32 leads to another performance improvement, beating a 1000-layer network of pattern `(16, 32, 64)`. (Parameter count is then `27` times the original value.)

* Comments
  * Stochastic depth works layer-wise, while Swapout works unit-wise. When a layer in Stochastic Depth is dropped, its whole forward- and backward-pass don't have to be calculated. That saves time. Swapout is not going to save time.
  * They argue that dropout+BN would also profit from using stochastic inference instead of deterministic inference, just like Swapout does. However, they don't mention using it for dropout in their comparison, only for Swapout.
  * They show that linear decay for their parameters (less dropping on early layers, more on later ones) significantly improves the results of Swapout. However, they don't mention testing the same thing for dropout. Maybe dropout would also profit from it?
  * For the above two points: Dropout's test error is at 5.87, Swapout's test error is at 5.68. So the difference is already quite small, making any disadvantage for dropout significant.


![Visualization](images/Swapout__visualization.png?raw=true "Visualization")

*Visualization of how Swapout works. From left to right: An input `x`; a standard layer is applied to the input `F(x)`; a residual layer is applied to the input `x + F(x)`; Skip Forward is applied to the layer; Swapout is applied to the layer. Stochastic Depth would be all units being orange (`x`) or blue (`x + F(x)`).*

