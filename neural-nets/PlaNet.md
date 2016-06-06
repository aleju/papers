# Paper

* **Title**: PlaNet - Photo Geolocation with Convolutional Neural Networks
* **Authors**: Tobias Weyand, Ilya Kostrikov, James Philbin
* **Link**: http://arxiv.org/abs/1602.05314
* **Tags**: Neural Network, geolocation, recurrent
* **Year**: 2016

# Summary

* What
  * They describe a convolutional network that takes in photos and returns where (on the planet) these photos were likely made.
  * The output is a distribution over locations around the world (so not just one single location). This can be useful in the case of ambiguous images.

* How
  * Basic architecture
    * They simply use the Inception architecture for their model.
    * They have 97M parameters.
  * Grid
    * The network uses a grid of cells over the planet.
    * For each photo and every grid cell it returns the likelihood that the photo was made within the region covered by the cell (simple softmax layer).
    * The naive way would be to use a regular grid around the planet (i.e. a grid in which all cells have the same size).
      * Possible disadvantages:
        * In places where lots of photos are taken you still have the same grid cell size as in places where barely any photos are taken.
        * Maps are often distorted towards the poles (countries are represented much larger than they really are). This will likely affect the grid cells too.
    * They instead use an adaptive grid pattern based on S2 cells.
      * S2 cells interpret the planet as a sphere and project a cube onto it.
      * The 6 sides of the cube are then partitioned using quad trees, creating the grid cells.
      * They don't use the same depth for all quad trees. Instead they subdivide them only if their leafs contain enough photos (based on their dataset of geolocated images).
      * They remove some cells for which their dataset does not contain enough images, e.g. cells on oceans. (They also remove these images from the dataset. They don't say how many images are affected by this.)
      * They end up with roughly 26k cells, some of them reaching the street level of major cities.
      * Visualization of their cells:
        ![S2 cells](images/PlaNet__S2.jpg?raw=true "S2 cells")
  * Training
    * For each example photo that they feed into the network, they set the correct grid cell to `1.0` and all other grid cells to `0.0`.
    * They train on a dataset of 126M images with Exif geolocation information. The images were collected from all over the web.
    * They used Adagrad.
    * They trained on 200 CPUs for 2.5 months.
  * Album network
    * For photo albums they develop variations of their network.
    * They do that because albums often contain images that are very hard to geolocate on their own, but much easier if the other images of the album are seen.
    * They use LSTMs for their album network.
    * The simplest one just iterates over every photo, applies their previously described model to it and extracts the last layer (before output) from that model. These vectors (one per image) are then fed into an LSTM, which is trained to predict (again) the grid cell location per image.
    * More complicated versions use multiple passes or are bidirectional LSTMs (to use the information from the last images to classify the first ones in the album).

* Results
  * They beat previous models (based on hand-engineered features or nearest neighbour methods) by a significant margin.
  * In a small experiment they can beat experienced humans in geoguessr.com.
  * Based on a dataset of 2.3M photos from Flickr, their method correctly predicts the country where the photo was made in 30% of all cases (top-1; top-5: about 50%). City-level accuracy is about 10% (top-1; top-5: about 18%).
  * Example predictions (using in coarser grid with 354 cells):
    ![Examples](images/PlaNet__examples.png?raw=true "Examples")
  * Using the LSTM-technique for albums significantly improves prediction accuracy for these images.
