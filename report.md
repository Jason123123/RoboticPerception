## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---
[//]: # (Image References)

[image1]: ./acc.png
[image2]: ./test2.png
[image3]: ./test3.png

# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify). 
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)  
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.

I first convert ROS cloud to PCL format so that PCL library functions could be used. Here are the following steps:

1. outlier removal filter: scale factor is set to 0.05
2. voxel grid downsampling filter: LEAFSIZE = 0.005
I first set the LEAFSIZE to 0.01 but found many false detections in test3. So I made this value smaller to capture more features for recognition.
3. Passthrough filter: z axis range is set to (0.6, 1.3) so that objects and the table is retained. y axis range is set to (-0.5, 0.5) to filter out the drop boxes.
4. RANSAC Plane Segmentation: remove the table from the scene. max_distance is set to 0.01 so that table is removed and most part of the objects are kept.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  

In the clustering process, color information of objects is ignored. I used EuclideanClusterExtraction() to perform a DBSCAN cluster search on 3D point cloud. I set clustertolerance to 0.015, minclustersize to 20 and maxclustersize to 2500. If the maxsize parameter is too small, then we may not cluster big objects properly.

Each cluster is assigned a color to be visualized in RViz.

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
I implemented the `compute_color_histograms()` and `compute_normal_histograms()`. The binsize is set to 32 for a balance between accuracy and speed.

To train the model, I run the `capture_feature()` to extract features from each object. Each object is spwaned for 60 times to be viewed from different angles(I first spawn 5 times for each and got very bad accuracy). 

These features are used to train an linear SVM. The image below shows the training accuracy.
![alt text][image1]


### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

After all objects have been recognized, I called the `pr2_mover()` function. This function reads in several parameter tables which specify the dropbox location and where each object should go to. I converted these parameters into python dictionaries. I iterate over detected objects, compute the centroids of them and determine the dropbox positions they belong to. At last, such information is output as a .yaml file.


#### Discuss
My program achieves 100% accuracy in test1, 100% percent accuracy in test2 and 87.5%(7/8) accuracy in test3.

![image2]
![image3]


In this project, I used an linear SVM without much parameter tunning. To make the recognition pipeline work better, I think we can try using kernels or extracting more features to train the SVM.
