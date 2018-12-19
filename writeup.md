## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


[//]: # (Image References)

[image1]: ./imgs/test_world1_obj.PNG
[image2]: ./imgs/train_cnt1.PNG
[image3]: ./imgs/train_prob1.PNG
[image4]: ./imgs/test_world2_obj.PNG
[image5]: ./imgs/train_cnt2.PNG
[image6]: ./imgs/train_prob2.PNG
[image7]: ./imgs/test_world3_obj.PNG
[image8]: ./imgs/train_cnt3.PNG
[image9]: ./imgs/train_prob3.PNG


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
Refer to the code section in project_template.py.
First of all, the camera point cloud data we get contains noise.
The first pipeline in the image filtering is the statistical outlier filter.
To minimize the point cloud noise, I have used an outlier filter of mean(3) and standard deviation(0.1) for world 1&2, and mean(5) for world3. The rule of thumb for tweaking mean and std dev is that, the outlier noise points are further away from information points. After some trial-and-error, I settle at the forementioned mean and dev. Now, the filtered cloud looks much less noisy -- in fact, very clear and improves the overall performance instantly.

The downsampling is fairly straight-forward with a voxel size of 0.01 (meter).

I then used passthrough filter to filter out the areas outside of our interest. Specially, I applied passthrough filter about z, y, and x axis to only leave the table and objects in the scene (filtering out the bins using y axis, and shadows and desk rims using x axis) 

After the above steps, I applied RANSAC plane segmentation to extract the desktop, and objects, respectively.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  
Next up, is the segmentation of objects using KD-tree. The previously tuned parameters (in Exercises1-3) for min/max cluster size and tolerance work effectively in this project.

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
After obtaining the object clusters, we load our previously trained SVM model to do object detection.
For world1&2, 15-20 feature captions are needed for each object to achieve a 100% detection rate.
For world3, I had to use 45 feature captions to train the SVM, resulting in a 100% detection rate. The reason is that we have more objects, and some objects are blocked and have very limited view.

### Results
#### World 1 object detection 
![demo-1][image1]
World 1 object SVM training confusion matrix 
![svm-1][image2]
![svmp-1][image3]
#### World 2 object detection
![demo-2][image4]
World 2 object SVM training confusion matrix 
![svm-2][image5]
![svmp-2][image6]
#### World 3 object detection
![demo-3][image7]
World 3 object SVM training confusion matrix
![svm-3][image8]
![svmp-3][image9]

### Observations & Takeaways
1. Sometimes, the SVM train data can be 'remembered' in the environment and can deteriorate the detection. For example, after running several trains and perceptions, the object detection got messed up and detects everything as 'soap2' or 'biscuits'. The solution to this is to close all terminal, even restart VM, to start a fresh training, and the data will be normal again. For this reason, I saved the training_set.sav and model.sav with each world that I trained. Also, the capture_features.py is saved with each world since they vary slightly.
2. The statistical outlier filter is a huge performance boost. Once I was able to get most the noise out, the detection becomes really easy. I hardly need to tune any filter paramters following the passthrough filter.
3. The SVM normalized confusion matrix may not be an accurate indicator of how well the model can do, especially for world3, since many objects are blocked and only a small patch of view is available.
4. I was able to achieve a good detection rate with simply adding more feature scans of each object for this project. In Exercise 3, I have tried to use a different SVM hyperplane but the results are not good. I have also tried increasing the bin size from 32 to 64, but it has made the results worse.

### Improvements & Future work
Since only a front view is available to the camera, one thing that can be improved is to capture more of the front view features of objects. Another way is to have more camera views such as when the robot moves to scan the collision map. However, as the objects got picked, this problem should alleviate.

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

Due to time constraints, I've yet to get to the pick and place part, so, I'll play offline and update when I can!
Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.  



