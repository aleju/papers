# Paper

**Title**: Fractional Max-Pooling
**Authors**: Benjamin Graham
**Link**: http://arxiv.org/abs/1412.6071
**Tags**: Neural Network, Pooling
**Year**: 2015

# Summary



-------------------------

# Rough chapter-wise notes

* (1) Convolutional neural networks
  * Advantages of 2x2 max pooling (2MP): fast; a bit invariant to translations and distortions; quick reduction of image sizes
  * Disadvantages: "disjoint nature of pooling regions" can limit generalization (i.e. that they don't overlap?); reduction of image sizes can be too quick
  * Alternatives to 2MP: 3x3 pooling with stride 2, stochastic 2x2 pooling
  * All suggested alternatives to 2MP also reduce sizes by a factor of 2
  * Author wants to have reduction by sqrt(2) as that would enable to use twice as many pooling layers
  * Fractional Max Pooling = Pooling that reduces image sizes by a factor of `1 < alpha < 2`
  * FMP introduces randomness into pooling (by the choice of pooling regions)
  * Settings of FMP:
    * Pooling Factor `alpha` in range [1, 2] (1 = no change in image sizes, 2 = image sizes get halfed)
    * Choice of Pooling-Regions: Random or pseudorandom. Random is stronger (?). Random+Dropout can result in underfitting.
    * Disjoint or overlapping pooling regions. Results for overlapping are better.

* (2) Fractional max-pooling
  * For traditional 2MP, every grid's top left coordinate is at `(2i-1, 2j-1)` and it's bottom right coordinate at `(2i, 2j)` (i=col, j=row).
  * It will reduce the original size N to 1/2N, i.e. `2N_in = N_out`.
  * Paper analyzes `1 < alpha < 2`, but `alpha > 2` is also possible.
  * Grid top left positions can be described by sequences of integers, e.g. (only column): 1, 3, 5, ...
  * Disjoint 2x2 pooling might be 1, 3, 5, ... while overlapping would have the same sequence with a larger 3x3 grid.
  * The increment of the sequences can be random or pseudorandom for alphas < 2.
  * For 2x2 FMP you can represent any alpha with a "good" sequence of increments that all have values `1` or `2`, e.g. 2111121122111121...
  * In the case of random FMP, the optimal fraction of 1s and 2s is calculated. Then a random permutation of a sequence of 1s and 2s is generated.
  * In the case of pseudorandom FMP, the 1s and 2s follow a pattern that leads to the correct alpha, e.g. 112112121121211212...
  * Random FMP creates varying distortions of the input image. Pseudorandom FMP is a faithful downscaling.
 
* (3) Implementation
  * In their tests they use a convnet starting with 10 convolutions, then 20, then 30, ...
  * They add FMP with an alpha of sqrt(2) after every conv layer.
  * They calculate the desired output size, then go backwards through their network to the input. They multiply the size of the image by sqrt(2) with every FMP layer and add a flat 1 for every conv layer. The result is the required image size. They pad the images to that size.
  * They use dropout, with increasing strength from 0% to 50% towards the output.
  * They use LeakyReLUs.
  * Every time they apply an FMP layer, they generate a new sequence of 1s and 2s. That indirectly makes the network an ensemble of similar networks.
  * The output of the network can be averaged over several forward passes (for the same image). The result then becomes more accurate (especially up to >=6 forward passes).

* (4) Results
