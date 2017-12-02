# Paper

* **Title**: Computer Vision for Autonomous Vehicles: Problems, Datasets and State-of-the-Art
* **Authors**: Joel Janai, Fatma Güney, Aseem Behl, Andreas Geiger
* **Link**: https://arxiv.org/abs/1704.05519v1
* **Tags**: Neural Network, self-driving, overview, RCNN, segmentation, tracking, LiDAR, stereo, depth, CRF
* **Year**: 2017

# Summary

* What
  * They give an overview of the current state of the art for self-driving cars.

* How
  * (Nothing here, it's just an overview article.)

* Results
  * They focus their overview on autonomous vision, i.e. on components/models that use
    computer vision to analyze images and not on models that make the final driving/steering
    decisions.
  * **History of Autonomous Driving**
    * ITS = Intelligent Transportation Systems
    * PROMETHEUS was an early (1986) european project to research self-driving vehicles.
      * In 1995 they drove from Munich to Odense (Denmark), being autonomous 95% of the time.
    * Navlab (1988) was a similar US-american project. They completed an autonomous drive from Pittsburgh, PA to San Diego, CA in 1995.
    * US launched National Automated Highway System Consortium (NAHSC) in 1995.
    * Japan launched Advanced Cruise Assist Highway System Research Association in 1996.
    * Pomerleau & Jochem (1996) drove from Washington DC to San Diego, being autonomous 98% of
      the time (only steering, not accelerating/decelerating).
    * Towards 2000, simple methods (e.g. lane following) became better and computational power was
      increasingle available, but things like reflections, wet roads or direct sunlight still
      caused problems.
    * Franke et al. (2000) suggested a first system to drive in urban environments,
      including depth-based obstacle detection and traffic sign recognition.
    * Prototype cars developed from VisLab approaches during 2000-2010 are ARGO, TerraMax, BRAiVE.
    * Bertozzi et al. (2011) drove semi-autonomous from Italy to China. Their system e.g. could
      detect lane markings or preceding cars and could perform e.g. leader-following and stop-and-go.
    * Broggi et al. (2015) drive in an urban environment in Parma, including e.g. roundabouts,
      traffic lights and intersections.
    * Furgale et al. (2013) drive autonomously with a car using close-to-market sensors and using
      vision-only navigation.
    * Google started their self-driving car project in 2009 and drove 1.5 million miles
      autonomously (as of march 2016). They split the project off into Waymo in 2016.
    * Tesla rolled out a semi-autonomous system in 2015.
    * Ziegler et al. (2014) drove from Mannheim (Germany) to Pforzheim (Germany) nearly fully
      autonomous.
    * Bojarski et al. (2016) drove from Holmdel to Atlantic Highlands (NJ), being autonomous 98% of
      the time.
    * Competitions
      * ELROB is a european competition, focusing on military applications and rough terrains.
      * DARPA Grand Challange 2004 offered one million for the first team to drive fully
        autonomously a 150 mile route between California and Nevada. (First teams accomplished that
        in 2005.)
      * DARPA Urban Challange 2007 similarly focused on urban driving, including other traffic and traffic
        regulations.
      * GCDC 2011 and 2016 in the Netherlands focused on convoy driving.
  * **Datasets and Benchmarks**
    * KITTI and Cityscapes are the most well-known datasets for autonomous vehicles.
    * Currently, datasets with several thousand labeled examples are state of the art.
    * Real-world datasets are often needed, but usually require tedious manual labeling.
      Mechanical Turk eases that problem a bit, but label quality tends to be low.
    * Stereo/3D Reconstruction datasets
      * Middlebury stereo benchmark: Multi-frame stereo data for stereo matching algorithms, pixel-level labels (partly hand-annotated, partly algorithmically generated)
      * Middlebury v3: More data added to Middlebury with labels of higher quality.
      * Middlebury multi-view stereo (MVS) benchmark: Calibrated multi-view image datset for MVS approaches. Contains two scenes.
      * TUD MVS dataset: Similar to Middlebury MVS, but with 124 scenes. Labels generated via structured light.
      * Schöps et al. (2017): High-resolution DSLR images synchromized with low-resolution stereo videos, annotated via high-resolution lasers.
    * Optical Flow datasets
      * Middlebury flow benchmark: Non-rigid, synthetic motion sequences with labels automatically derived using hidden fluorescent textures painted onto the objects. Limited size, small movements, low lighting/shadow variation.
      * Janai et al. (2017): 160 diverse real world scenes (1280x1024 resolution). Labels automatically generated using pixelwise tracking and high-speed cameras.
    * Object Recognition and Segmentation datasets
      * ImageNet
      * PASCAL VOC: Benchmark for object classification/detection/segmentation and action recognition. Real world images with high variance. Up to 20 classes, 11k images, 27k bounding boxes.
      * MS COCO: For object detection, instance segmentation, contextual reasoning. 91 classes, 2.5M instances, 328k images.
      * Cityscapes
      * TorontoCity
    * Tracking datasets
      * MOTChallenge: 14 real world video sequences for pedestrian tracking. Cameras static and moving. Evaluation measures are Multiple Object Tracking Accuracy (MOTA) and Multiple Object Tracking Precision (MOTP).
    * Aerial Image datasets
      * ISPRS: Aerial images. For urban object detection and 3d building reconstruction.
    * Autonomous Driving datasets
      * KITTI
        * Dataset for stereo vision, optical flow estimation, visual odometry/SLAM, 3D object detection
        * Captured via car. 6 hours of driving. High-resolution color camera, grayscale stereo cameras, 3D laser scanner, high-precision GPS/IMU inertial navigation system.
        * For stereo vision and optical flow: 194 train + 195 test images (1280x376). Only static scenes with moving camera.
        * Additional annotations for moving objects in 400 dynamic scenes by Menze & Geiger (2015).
        * For visual odometry / SLAM: 22 stereo scenes, length 39.2km.
        * For object detection: 7481 train + 7518 test images with annotated 3d bounding boxes. Vehicles, pedestrians, cyclists.
        * For road/lane detection: 600 diverse training and test images by Mattyus et al. (2016).
        * ![KITTI](images/Computer_Vision_for_Autonomous_Vehicles_Overview/kitti.jpg?raw=true "KITTI")
      * HCI benchmark: 28k steoreo images with optical flow and stereo ground truth. Low diversity.
      * Caltech Pedestrian Detection Benchmark: 250k frames recorded from vehicle. 350k bounding boxes of 2.3k unique pedestrians. With temporal correspondence and occlusion labels.
      * Cityscapes
        * Stereo image sequences recorded from a car (real world street scenes).
        * Annotations for pixel-level and instance-level semantic labeling.
        * 5k images with detailed annotations, 20k with coarse annotations.
      * TorontoCity
        * Building height estimation, road centerline and curb extraction, building instance segmentation, building contour extraction, semantic labeling, scene classificiation.
        * Images from airplanes, drones, cars. Covers the Toronto area.
    * Long-Term Autonomy datasets
      * Maddern et al. (2016): Images, LiDAR, GPS data from 1k km driving in Oxford, spread over one year.
    * Synthetic datasets
      * MPI Sintel: Optical flow for 1.6k frames from the Sintel movie.
      * Dosovitskiy et al. (2015): 2D images of flying chairs in front of random background images. For optical flow estimation.
      * Mayer et al. (2016): 2D images of flying things in front of random background images. Also images of car models in front of houses and trees. For optical flow estimation.
      * Virtual KITTI: 35 videos with 17k high resolution frames for object detection, tracking, semantic/instance segmentation, depth, optical flow. Varying environmental conditions (e.g. rain). Generated using a game engine.
      * SYNTHIA: 13.4k images and 4 videos with 200k frames for semantic segmentation in urban scenarios. Generated using Unity Engine.
      * Richter et al. (2016): 25k images from GTA5 with dense semantic annotations.
      * Qiu & Yuille (2016): A method to create virtual worlds using Unreal Engine 4 and how to connect them with Caffe.
  * **Camera Models and Calibration**
    * Calibration
      * Calibration = estimate intrinsic & extrinsic parameters of sensors (e.g. range sensors, fish-eye cameras, ...)
      * This allows to relate 2D image points with 3D world points.
      * Calibration is done with the help of fiducial markings on checkerboard patterns.
      * Desired features of good calibration systems are: high accuracy, speed, robustness, minimal assumptions
    * Omnidirectional Cameras
      * Omnidirectional Cameras = cameras with 360 degrees field of view
      * Obviously good for self-driving cars, give full situational awareness.
      * Catadioptric Cameras = combine standard camera and mirror (to generate 360 degrees field of view)
      * Dioptric Cameras = Use doptric fisheye lenses
      * Polydioptric Cameras = Use multiple cameras with overlapping field of view
      * Omnidirectional Cameras can be differentiated by the projection center: Central, noncentral
      * For central cameras, all light rays intersect in a single point in 3D. Benefit: Geometrically correct generated images => allows to use epipolar geometry on the images.
      * Distortion in omnidirectional cameras is too high to perform calibration based on simple, linear projections (as opposed to pinhole cameras).
      * Omnidirectional cameras are roughly comparable to LiDAR (radar systems), but are far cheaper, perceive color and do not suffer from rolling shutter effects.
      * Häne et al. (2014) predict dense depth maps from omnidirectional cameras in real time (GPU) based on the plaen-sweeping stereo matching algorithm.
    * Event Cameras
      * Event Cameras = Produce an event when a brightness change passes a threshold (sparse data); they work at microsecond level.
      * Interesting for self-driving cars due to their very high speed (low reaction times).
      * Event Cameras also do not suffer from motion blur and have high dynamic range.
      * Data is difficult to handle due to algorithms being adapted towards full images.
      * Events can be accumulated over time to generate full images, but this runs contrary to the speed benefit.
  * **Representations**
    * Pixel-based
      * Each pixel is a random variable. E.g. in CNNs or PGMs.
      * Most fine-grained.
      * High complexity.
    * Superpixels
      * Derived from segmentation of images into atomic regions with similar color/texture.
    * Stixels
      * Intended for 3D traffic scenes.
      * Somewhere between pixels and objects.
      * Each stixel is a vertical stick in the landscape.
      * Each stixel is defined by its position relative to the camera and its height.
      * The width is constant.
      * They are good for representing obstacles in front of the car.
      * ![stixel](images/Computer_Vision_for_Autonomous_Vehicles_Overview/stixel.jpg?raw=true "stixel")
    * 3D Primitives
      * Represent 3D geometry with primitives.
      * Significantly cheaper computationally.
  * **Object Detection**
    * Useful to detect other participants and obstacles.
    * Sensors
      * Cameras are cheapest.
      * For daytime, just ordinary cameras, i.e. visible spectrum (VS) cameras.
      * For nightime, use thermal infrared (TIR) cameras, which captures temperature differences.
      * Lasers can be used to detect range information, useful for 3d object detection and localization.
      * Weather effects can influence sensors.
      * Sensor fusion combines information from different sensors.
    * Standard Pipeline
      * Preprocessing => RoI extraction ("regions of interest") => classification per RoI => Verification/Refinement
      * Preprocessing: E.g. exposure/gain adjustment, camera calibration, ...
      * RoI extraction: E.g. by sliding window
      * Classification: Classify each RoI (e.g. cascade of weak classifiers in classic Viola Jones)
      * Part-based Approaches: Develop one model per part of an object (simpler to learn per part), e.g. in Deformable Part Model (DPM)
    * 2D Object Detection
      * Popular datasets for 2D object detection in autonomous cars: KITTI, Caltech-USA
      * Task is dominated currently by CNNs, mainly R-CNN variations.
      * KITTI contains a lot of small examples, but R-CNNs tend to be bad at detecting these.
        Therefore many current methods focus on how to improve R-CNNs for small bounding boxes.
    * 3D Object Detection from 2D Images
      * Not much here, still behind 2D detectors.
    * 3D Object Detection from 3D Point Clouds
      * LiDAR laser range sensor data is usually sparse and with limited spatial resolution.
      * State of the art is behind in accuracy to imag based methods.
      * Currently best method on KITTI uses the LiDAR's point cloud from bird view and camera RGB images as a CNN input and then uses a RoI-Pooling based method for detection, similar to 2D bounding box detection.
    * Person Detection
      * People are common obstacles for self-driving cars and are hard to detect due to high variation in poses and clothes.
      * Pedestrian Protection Systems (PPS) detect people around the car and warn the driver.
        These systems have to be basically flawless.
      * Modern pedestrian detectors use CNNs.
      * Temporal Cues
        * Adding temporal information to object detectors improves results.
        * For pedestrians e.g. dynamic gait, motion parallax.
      * Scarcity of Target Class
        * Positive examples of pedestrians are much rarer in example images than negative ones.
        * Enzweiler & Gavrila (2008) suggest to use a generative model to generate synthetic examples and train a classifier on these.
      * Real-time Pedestrian Detection
        * Stixel world representations can be used to reduce the search space, resulting in high performance.
    * Human Pose Estimation
      * Pose estimation is useful to e.g. estimate the intentions of a person.
      * Previously done with two-staged approach: First detect body parts, then estimate pose.
      * Now done with CNNs.
    * Discussion
      * Detection of large and obvious objects works well.
      * Smaller and occluded objects cause problems.
      * Differentiating pedestrians and cyclists causes problems.
      * Crowds of people can cause problems.
  * **Semantic Segmentation**
    * Semantic Segmentation = Assign each pixel in an image a class label
      * ![cityscapes](images/Computer_Vision_for_Autonomous_Vehicles_Overview/cityscapes.jpg?raw=true "cityscapes")
    * Previously done via MAP in CRFs (input either pixels or superpixels).
    * But not efficient and few simple features could be used.
    * Higher-order features with long-range interactions were developed.
    * More efficient inference algorithms were developed.
    * Fully convolutional networks succeeded CRFs for semantic segmentation.
    * The networks face the challange to both reason on a coarse level as well as assigning classes on a fine level (pixel-wise).
      * Solved e.g. with dilated convolutions.
      * SegNet memorizes the indices of max-pooling cells to reverse the process during upsampling (saves parameters for upsampling steps).
      * Using skip connections between resolutions is also an option.
      * Another option is to upsample lower scales and then fuse scales of various resolutions in additional convolutions.
      * Residual architectures can also help.
    * CNNs have also been combined with CRFs. Usually, the CNN is executed first and the CRF then refines the predictions at the pixel-level.
    * Currently, models perform semantic segmentation well at the coarse level, but have more problems at the fine level.
    * Semantic Instance Segmentation
      * Semantic Instance Segmentation = Detect individual objects, classify them and segment them
      * Two approaches: Proposal-based Instance Segmentation and Proposal-free Instance Segmentation
      * Proposal-based Instance Segmentation first predicts region proposals (e.g. bounding boxes) and then segments each object within the region proposal. Works similarly to R-CNNs.
      * Proposal-free Instance Segmentation predicts instance-level segmentation and class-level segmentation in one step.
      * Models are currently much weaker at instance level segmentation than semantic segmentation.
    * Label Propagation
      * Annotations for Semantic/Instance Segmentation are very labour intensive to produce.
      * One possible solution is to leverage information from previous/future frames to simplify the annotation process.
      * I.e. annotate few keyframes, then propagate the annotations to the frames in between, based on e.g. pixelwise color information.
    * Semantic Segmentation with Multiple Frames
      * When multiple frames are available, temporal information can be used to improve segmentation accuracy.
    * Semantic Segmentation of 3D Data
      * Segmentation can also be performed in 3D space, instead of 2D.
      * This is less common, but probably more useful for self-driving cars.
      * E.g. done by 3D CNNs in voxel space. This can be expensive with regards to computation and memory. Riegler et al. (2017) suggest to operate in an Octree space, thereby exploiting the sparseness of the output.
    * Semantic Segmentation of Street Side Views
      * One subtask of semantic segmentation is to segment street side views, e.g. views of buildings.
      * This is e.g. useful for navigation/localization.
    * Semantic Segmentation of Aerial Images
      * Used to find urban objects in aerial/satellite images.
      * Useful e.g. for navigation - even in remote areas and in case of very recent changes.
      * Difficult, because the objects have heterogeneous appearances that also can't be expressed with simple priors.
      * Lots of CRF/MRF solutions available.
      * Aerial images are usually of low resolution (e.g. 1 meter = 1 pixel), whereas ground-based images are of high resolution.
      * Some solutions therefore combine aerial and ground based images, e.g. using a joint MRF.
      * The ISPRS segmentation challange contains data for (urban) semantic segmenation of airborne sensors.
      * The best solutions on ISPRS use CNN+CRF or just CNNs.
    * Road Segmentation
      * Correct road segmentation obviously important for self-driving cars.
      * Again, CNN solutions available.
      * Some solutions to decrease labeling effort. E.g. one maps OpenStreetMap annotations onto the image using GPS coordinates and car pose.
      * Free Space Estimation is a subfield of Road Segmentation
        * Deals with detecting cars and other objects that stick out of the ground or somehow block the self-driving car.
        * Solutions often employ stereo cameras for depth estimation.
        * Long Range Obstacle Detection can also use radars, as their error does not increase quadratically in depth (as opposed to stereo cameras).
  * **Reconstruction**
    * Stereo
      * Stereo estimation = extract 3D information (i.e. depth) from 2D stereo camera images
      * Stereo cameras consist of two cameras mounted next to each other.
      * Stereo estimation algorithms compute correspondences between images taken by the two cameras at the same points in time.
      * Stereo estimation is useful for 3D reconstruction or e.g. free space estimation
      * Taxonomies
        * Feature-based vs. area-based methods: Feature-based methods provide sparse edge-based depth maps, area-based methods generate dense outputs.
        * Local vs global: Local methods compute image disparities based on best matches (winner takes all), global methods use energy-minimization.
      * Matching Cost Function
        * Used for matching points between left and right camera images.
        * Usually assume that both cameras are rectified, reducing search space to horizontal line.
        * Cost function computes cost for each pixel and possible disparity. Should take lowest values at true disparity.
        * Algorithms make assumption of constant appearance between matching points. Strong lighting changes can cause problems.
      * Semi-Global Matching (SGM) is a high-speed high-accuracy method for stereo estimation. It is optionally performed on top of CNN features. It is theoretically similar to belief propagation.
      * Using sensible priors/regularization helps with ambiguities in depth estimation.
      * Total Variation is one used prior.
      * CNNs are nowadays common and fast for stereo estimation.
      * E.g. one can embed both left and right images into a feature space and then compute the correlation between the two (when shifting an image an the x-axis).
        * ![stereo matching](images/Computer_Vision_for_Autonomous_Vehicles_Overview/stereo_matching.jpg?raw=true "stereo matching")
    * Multi-View 3D Reconstruction
      * Deals with reconstructing 3D geometry from many images.
      * Also makes use of sensible priors.
      * Methods usually differ by: form of photo-consistency function, scene representation, visibility computation, priors, initialization requirements.
      * Scene representation can be: Depth map, point cloud, (triangular) mesh/surface, volumetric/voxels
      * Common algorithm for depth maps: Plane Sweeping Stereo algorithm
      * Common algorithm for point clouds: Patch-based Multi-View Stereo (PMVS)
      * A voxel grid can be converted to surfaces using the Marching Cubes algorithm.
      * Urban reconstruction algorithms: Methods to automatically reconstruct the 3D geometry of cities (here from images gathered by self-driving cars).
      * Most common data for multi-view reconstruction are ground based images (i.e. from cars), aerial/satellite images, LiDAR scans. Aerial is readily available but coarse.
    * Reconstruction and Recognition
      * Performing 2D semantic segmentation and 3D reconstruction in one step/model should be beneficial for the accuracy of both tasks, e.g. all pixels belonging to cars can be expected to have certain possible geometric shapes.
        * ![joint 3d scene reconstruction](images/Computer_Vision_for_Autonomous_Vehicles_Overview/joint_3d_scene_reconstruction.jpg?raw=true "joint 3d scene reconstruction")
      * Can be done on small scale (single buildings) or large scale (whole cities).
      * Large scale is difficult due to memory constraints. (Notice also that labels may increase, potentially increasing memory demands even further.) Octrees can help.
      * Priors for 3D shapes can be derived e.g. using PCA or Gaussian Process Latent Vriable Models (GP-LVM).
  * **Motion & Pose Estimation**
    * 2D Motion Estimation - Optical Flow
      * Optical Flow = two dimensional motion of brightness patterns between two images
      * Useful e.g. for tracking of objects, ego-motion estimation, structure-from-motion
      * Optical flow predicts motion vectors. Each vector has two components. Two constraints are used: (a) the brightness of each pixel over time is assumed to be constant, (b) movements are assumed to be smooth between neighbours.
      * Occlusions, large displacement and fine details are still challenging today.
      * Pixel intensity is assumed to be constant over time, but that is not always the case due to e.g. reflections or transparency.
      * Original optical flow method was based on variational formulation. It assumed pixel intensity to be constant and added a smoothness constraint.
      * Later algorithms used more robust methods than just assuming constant pixel intensity.
      * A common approach is to first estimate the optical flow at a coarse level and then refine towards finer levels.
      * Some methods use nearest neighbour search of 2D patches, optionally in a high-dimensional feature space.
      * Many accurate optical flow algorithms are accurate but far too slow for self-driving cars. Some newer methods based on CNNs though perform well.
      * Training of optical flow algorithms often happens based on Sintel and KITTI datasets.
      * Comparison of methods on KITTI happens based on absolute endpoint error (EBE). An error is any predicted motion vector which's endpoint is more than 3 pixels or 5% of the image dimension away from its true endpoint. The errors are measured in percent of all vectors (i.e. lower is better) and are split into background, foreground and other regions. The final EBE is the average of these error categories.
      * For self-driving cars, it can be sensible to predict optical flow as following epipolar lines, radiating from the center of the image.
      * Another optimization is to first perform instance segmentation and then predict optical flow per instance.
    * 3D Motion Estimation - Scene Flow
      * Scene flow = optical flow for 3D scenes
      * Usually by predicting motion vectors on surfaces of objects.
      * Dense prediction preferred, but sparse prediction is faster.
    * Ego-Motion Estimation
      * Ego-Motion Estimation = Estimate position and orientation of the car
      * Wheel odometry was previously used for that: Estimate position and orientation of the car by measuring how much the wheels spinned. This causes problems e.g. on when the wheels are slipping on wet surfaces.
      * Visual/LiDAR-based odometry: Estimate position and orientation of the car by using images/LiDAR. More robust and can recover from errors by recognizing visited places (loop closure).
      * Visual odometry estimates the changes between two images and then accumulates these over time.
      * Visual odometry can be split into feature-based and direct formulations.
      * Feature-based formulations first convert images into feature spaces, direct formulations work with the raw input data.
      * Feature-based formulations often use keypoint extraction, which can be problematic with street images due to their lack of corners. This makes direct formulations usually more accurate.
      * Drift
        * Predicting changes between pairs of images (incremental approach) tends to accumulate errors over time (drift).
        * This is fixed by an iterative refinement (often computationally expensive). E.g. done by minimizing errors of 3D points or by using information from SLAM (simultaneous localization and mapping) used to recognize known locations (loop closure).
      * KITTI is the only large dataset available for visual odometry.
      * Monocular methods for visual odometry perform worse than methods that use 3D data.
      * LiDAR-based odometry outperforms monocular visual odometry for ego-motion estimation. Stereo cameras though perform almost as good as LiDARs.
      * Current solutions for ego-motion estimation have mostly problems crowded highways, narrow streets and when strong turns are taken.
    * Simultaneous Localization and Mapping (SLAM)
      * SLAM = Navigate through an environment and create a map of it at the same time
      * SLAM must be able to deal with changing environments.
      * In case of autonomous vehicles it also has to deal with large scale environments and real-time application.
      * Early solutions used extended kalman filters or particle filters.
    * Localization
      * Localization = Determine position in the world
      * Can be indoor or outdoors.
      * Measurements might be noisy.
      * Is a subroutine of SLAM (for loop closure detection and drift correction).
      * Used sensors are GPS and camera (visual inputs).
      * GPS is only accurate to about 5m (sometimes more accurate, but not reliable).
      * Visual detection often works based on recognizing landmarks ("place recognition") and requires some kind of similarity measure.
      * Monte Carlo methods are sometimes used for visual localization.
      * Visual localization can be split into: Topological methods, metric methods.
      * Topological localization is coarse but reliable. (Predicts position from a finite set possible positions.)
      * Metric localization is fine but unreliable (especially for longer journeys).
      * Topometric localization: Mixture of topological and metric localization.
      * Current methods are on the order of 1m accuracy on short known routes or 4m accuracy on larger areas.
      * Structure-based Localization: Predict (camera) location as well as all camera parameters (orientation, intrinsics) simultaneously.
      * Cross-view Localization: Match ground based images (rare but detailed) with aerial/satellite images (easy to get but coarse). This allows to perform geo-localization.
      * One way to do cross-view localization is to do (1) apply a CNN to ground based images to convert them to features, (2) apply a CNN to aerial images showing the same locations as the ground based images to convert them to features, (3) train networks from step 1 and 2 to produce as similar features as possible. Then do feature matching.
  * **Tracking**
    * Tracking = Estimate state of one/multiple objects over time given measurements from sensor
    * State = location, velocity, acceleration (usually)
    * Useful e.g. for automatic distance control or to anticipate driving maneuvers of other traffic participants
    * Prediction of the behaviour of pedestrians and bicyclists is difficult, as they can abruptly change it.
    * Difficulties: Variation of appearance within the same class (e.g. different poses of pedestrians), occlusions (possibly by objects of the same class), strong lighting, reflections in mirrors/windows.
    * Tracking is often formulated as a bayesian inference problem. Goal: Estimate posterior probability density function of current observation, given previous state. Updated recursively.
    * E.g. implemented via extended kalman filters and particle filters.
    * Recursive formulations have problems with recovering from errors.
    * Non-recursive formulations have gained popularity because of that. These optimize energy functions with respect to all trajectories in a temporal window. The search space for this can be large and should be somehow minimized.
    * Datasets for multi object tracking are PETS (static camera), TUD (static), MOT (dynamic), KITTI (dynamic).
  * **Scene Understanding**
    * Scene understandig = A full understanding of the surrounding area of a self-driving car
    * Involves many subtasks, such as depth estimation, object detection, ...
    * Specifically challenging are urban and sub-urban scenes: Many independently moving objects, high variance in appaerance of objects along the road, more variance in road layouts, more difficult lighting.
    * One solution uses street view images and combines them with information from OpenStreetMap. This way, the authors can automatically derive for the street view images e.g. the number of lanes, the road curvature or the distance up to the next intersection and train the model predict these information.
  * **End-to-End Learning of Sensorimotor Control**
    * Traditional systems for self-driving cars contain many components, e.g. for object detection (traffic signs, pedestrians, ...) or tracking.
    * These components' predictions are then combines using some kind of rule based system in order to create driving decisions.
    * Modern system though lean more towards end-to-end learning, where the model e.g. gets the front facing camera's image and ground truth steering decisions and learns this way to steer.
    * One method also uses inverse reinforcement learning to extract the unknown reward function (which rewards correct steering decisions).


