[//]: # (Image References)
[pr2_robot]: ./misc/PR2.png
[raw_point_cloud]: ./Exercise-1_Images/Raw_Point_Cloud.png
[voxel_downsampled]: ./Exercise-1_Images/Voxel_Downsampled.png
[passthrough_filtered]: ./Exercise-1_Images/Passthrough_Filtered.png
[ransac_inliers]: ./Exercise-1_Images/RANSAC_Inliers.png
[ransac_outliers]: ./Exercise-1_Images/RANSAC_Outliers.png
[segmented_objects]: ./Exercise-2_Images/pcl_objects_cloud.png
[world1_conf_matrix_norm]: ./Confusion_Matrices/Final_Project_Conf_Matrix_World1_1.png
[world1_conf_matrix_not_norm]: ./Confusion_Matrices/Final_Project_Conf_Matrix_World1_2.png
[world1_labelled]: ./Misc_Images/World_1_Labelled.png
[world2_conf_matrix_norm]: ./Confusion_Matrices/Final_Project_Conf_Matrix_World2_1.png
[world2_conf_matrix_not_norm]: ./Confusion_Matrices/Final_Project_Conf_Matrix_World2_2.png
[world2_labelled]: ./Misc_Images/World2_Labelled.png
[world3_conf_matrix_norm]: ./Confusion_Matrices/Final_Project_Conf_Matrix_World3att2_1.png
[world3_conf_matrix_not_norm]: ./Confusion_Matrices/Final_Project_Conf_Matrix_World3att2_2.png
[world3_labelled]: ./Misc_Images/World3_Labelled.png

[![Udacity - Robotics NanoDegree Program](https://s3-us-west-1.amazonaws.com/udacity-robotics/Extra+Images/RoboND_flag.png)](https://www.udacity.com/robotics)

# 3D Perception Project with the Willow Garage PR2 Robot
---

## Introduction

This project focussed on perception and object recognition, for retrieval by a Willow Garage PR2 robot, with an RGBD camera following. The project followed a similar methodology to the [Amazon Robotics Challenge](https://www.amazonrobotics.com/#/roboticschallenge). Filters and clustering methods were used to isolate and recognise each object.

* The first stage of the perception pipeline is to reduce the number of points for computation using a voxel grid filter, to reduce the number of data points, and a passthrough filter, to reduce the visual area to only the area of interest (i.e. where the objects are located).

* The next stage of the perception pipeline used [RANSAC filtering](https://en.wikipedia.org/wiki/Random_sample_consensus) to isolate the objects from the table upon which they were placed.

* Clustering was then performed using the [DBSCAN algorithm](https://en.wikipedia.org/wiki/DBSCAN) in order segment each object point cloud for recognition.

* A [Support Vector Machine](https://en.wikipedia.org/wiki/Support_vector_machine) was then trained to recognise objects according to their colour histograms and thus recognise each segmented object from the DBSCAN segmentation.

## Filtering

### Voxel Downsampling, Passthrough Filter and RANSAC segmentation (Exercise 1 Pipeline Sections)

Before process the raw data from the RGBD camera needs processing.

Raw point cloud from Exercise 1:

![Raw Point Cloud][raw_point_cloud]

The first stage in the perception pipeline is to downsample to data from the raw XYZRGD data from the RGBD camera using a voxel grid filter. A leaf size of *0.01* was found to retain relevant detail while also significantly reducing the number of points in the point cloud for Worlds 1 and 2. Greater fidelity was found to improve recognition for World 3 were a leaf size of 0.002 was used.

![Voxel Downsampled Point Cloud][voxel_downsampled]

A passthrough filter was then applied along the z coordinate between *0.6* and *1.1* for exercise one to clip the data outside the range of interest for the perception task. For the final project the filter was tweaked further to between *0.61* and *0.9*.

![Passthrough Filtered Point Cloud][passthrough_filtered]

This data was then sent to a [RANSAC filter](https://en.wikipedia.org/wiki/Random_sample_consensus) which was used to identify and separate the objects themselves from the smooth plane of the table upon which they were placed.

![Isolated objects][ransac_outliers]

### DBSCAN segmentation (Exercise 2 Pipeline Sections)

The perception pipeline from exercise 1 was extended in exercise 2, using a DSCAN algorithm to segment each individual object point cloud based upon their mutual spacial proximity to each other.

The other segementation algorithm discussed in the lectures was the k-mean clustering algorithm, but this does not perform so well when the number of clusters presented is unknown. For the purposes of this project DSCAN was chosen as the clusters od f objects will already be spacially segmented but the number of objects will change.

A cluster tolerance of *0.001* was used with a minimum cluster size of *10* and a maximum of *250* for exercise two.

For Worlds 1 and 2 a cluster tolerance of *0.05* was used alonside a minimum cluster size of of *100* and a maximum of *3000*. For World 3 a cluster tolerance of *0.01* was found to perform well in addition to raising to maximum cluster size to *5000*.

![Segmented Objects from Exercise 2][segmented_objects]

### Feature Extraction, Support Vector Machine Training and Object Recognition (Exercise 3 Pipeline Sections)

Colour histograms are used to measure how each object looks when captured as an image. Each object is displayed in a series of random positions to give the model a 360-degree understanding of each object. HSV (Hue, Saturation, Value) colour distributions tend to perform better and more robustly under all light conditions than RGB colour data, so HSV values were used for the final perception pipeline, before being analysed as a histogram.

The number of bins decides how detailed the object model will be, but too many bins will overfit the data. 128 bins were chosen in this case for Worlds 1 and 2. 256 bins were found to be required for World 3.

The code for building the histograms is located in features.py.

capture_features.py saved the object features to a file named training_set.sav. It captured each object in 50 random orientations, using the HSV colour space and used 128 bins when creating the colour histograms.

## Project Outcomes

A support vector machine with a linear kernel and C values of between 0.05 and 0.1 were found to yield the most accurate confusion matrices for final project outcome delivery. 50 kfold cross validation was then used.

### World 1

#### Confusion Matrices

Non Normalised

![Non Normalized Confusion Matrix][world1_conf_matrix_not_norm]

Normalised

![Normalized Confusion Matrix][world1_conf_matrix_norm]

#### Labelled Items in Situ

![World 1 Labelled][world1_labelled]

We can see that the World 1 Confusion Matrices are 98% accurate and that all three objects (100%) in World One are labelled correctly.

### World 2

#### Confusion Matrices

Non Normalised

![Non Normalized Confusion Matrix][world2_conf_matrix_not_norm]

Normalised

![Normalized Confusion Matrix][world2_conf_matrix_norm]

#### Labelled Items in Situ

![World 1 Labelled][world2_labelled]

We can see that the World 2 Confusion Matrices are roughly 95% accurate and that four out of five (80%) objects in World 2 are labelled correctly.

### World 3

#### Confusion Matrices

Non Normalised

![Non Normalized Confusion Matrix][world3_conf_matrix_not_norm]

Normalised

![Normalized Confusion Matrix][world3_conf_matrix_norm]

#### Labelled Items in Situ

![World 3 Labelled][world3_labelled]

We can see that the World 3 Confusion Matrices are roughly 95% accurate and that six out of eight objects (75%) in World 3 are labelled correctly. World 3 required extra calibration training with 100 captures per object, 256 HSV color histogram bins per object and iterative tweaking of the DBSCAN clustering hyperparameters, as discussed above, before the necessary accuracy to pass the project could be achieved.

These confusion matrices themselves were also not the most accurate of those produced when tampering with hyperparameters, but they produced a model that worked best in practise.

## Future Recommendations

In future an automated grid search would help to better optimise the correct hyperparameters for each scenario. Particular difficulty was found with World 3 and much time was spent optimising. Also separate SVMs were used for each test world scenario to achieve the accuracy necessary to pass the project, so the SVM was not universal in practice. An SVM alone is probably not enough for object recognition and ideally I would use a Fully Convolutional Neural Network or a hybrid of Machine Learning and Neural Network methods for this object recognition task.

In world 3 the glue object was stuck behind the book object. For many of the recognition attempts the glue was not being recognised at all. Where objects are blocking the line of sight of the RGBD camera a DBSCAN segementation method alone is probably not the most optimal. You need semantic understanding of object features that an FCN would be able to give.

Overall this project gave good insight into the basics of object recognition, but it's clear that there is still much room for improvement.
