## Project: Search and Sample Return


**Objective**  
To achieve a minimum of 60 % fidelity to the ground truth while mapping at least 40% of the environment. Identify at least 1 rock samples from the envoironement.

**Environemnt /Setup**
* Udacity Rover Simulation Roversim
* Udacity RoboND-Python-Starterkit

**Notebook Analysis**
The purpose of this project is to map a environment using a rover and ensure at least 60% fidelity to the ground truth while mapping aat least 40% of the environment. The evnvironemnt and rover is simulated in the "Udacity Rover Simulation Roversim". There are two mode in the simulation software: Training mode and Autonomous Mode. A perception pipeline has been created to analyze the picture capture from the camera on the rover and provide guide for the rover to be able to move autonomously.

"Training mode" has been used to record some data to test out the perception pipeline.

The perception pipeline steps are as below.
1. Read in image samples. Two images are selected here to represent navigable route, obstacle, and rock.
![iamge1](output/train_images.png "image samples")

2. Use perspective transform and warp perspective function from OpenCV to transform the image from first person view to a top down map view. (perspect_transform(img, src, dst))
![iamge2](output/warped_train_images.png "perspective trasnform and warped")

3. Use color threshold function to differentiate navigable route (light brown), obstacle (dark brown), and rock(yellow). RGB has been selected to filter out the objects by using lower limit and upper limit. (color_thresh(img, rgb_thresh=(160, 160, 160), rgb_thresh_max=(255,255,255)))
![iamge3](output/color_threshold_train_images.png "color threshold")

4. Convert the pixel to pixel location on world map to map the data onto the map coordinates. Mean angle is calculated for the navigable route and is plot out as an arrow.
![iamge4](output/rover_centric_train_images_2.png "world map update")

The perception pipeline is populated in the `process_image()` function and the function is then used to process each frame of image to produce a video of mapping navigable route, obstacle, and rock on the ground truth map data.
![iamge4](output/video-output.png "world map update")

---

### Autonomous Navigation and Mapping


##### Perception
 The first change was to again add the rgb_thresh_max to the color_thresh function.  In my perception_step function, I set the Rover.vision_image red channel to rock_sample color threshhold output, green channel for walls, and blue channel for navigable terrain. I then converted all three of the threshholded outputs into rover-centric coordinates.  The navigable terrain and wall coordinates were transformed to world coordinates, and applied as follows:
 
 Blue channel of worldmap receives += 255 for all navigable terrain and -= 255 for walls

 Red channel of worldmap receives += 255 for all walls and -= 255 for navigable terrain
 
 I found that this method gave me the highest fidelity.  As for the rover-centric rock sample coordinates, they were used to check if there is a sample in the image, and determine the angle the rover needs to turn to get to that sample.  If a sample was in sight, the rovers mode was set to 'rock_visible', a custom mode I handle in the decision_step.  Further, if no rock sample is present, the navigable terrain rover-centric coordinates are converted to polar coordinates, and set to the rovers nav_dists and nav_angles variables.

##### Decision
In the decision_step function, I added a conditional to check if the rover is currently picking up a sample, and if so, to stop any movement and set the mode to 'stop'.  Next I added the check if a sample is near the rover.  If there is, then send the pickup sample signal to the rover, increment the samples_found count, stop moving, and set the mode to 'stop'.  
The last conditional I added was a check if the rover mode was 'rock_visible'.  If it was, then max out the throttle and steer toward the rock, regardless if there is not enough navigable terrain to continue much farther.  This was done because the rock samples are close to the walls, and the normal 'forward' mode tells the robot to stop and turn away from walls.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

My simulator was running at ~30 FPS with 1280x768 resolution and 'Beautiful' graphics quality.

My approach is working for the most part, and is able to map a good percentage of the environment it covers with ~85 % fidelity. I wasn't able to test the simulator for longer than a couple minutes due to a memory leak in the simulator. My rover can also successfully see and pickup rock samples, but it does not yet return them to its original position.  I could have included that the 'easy' way, waiting for the robot to randomly enter its starting position, but I would like to implement it in a more correct way in the future.  

A few drawbacks of my approach include that there is no intelligent algorithm for exploration of the map, so the rover will cover the same spots several times, and miss some spots where the mean angle does not take it.  There is also nothing to prevent the rover from driving in a circle for a while when following the mean angle of navigable terrain.  The last caveat of my rover is that in the case of getting stuck, there is no 'get unstuck' routine.

I would like to revisit this project in the future, and try some end to end deep learning for this rover.  I would also like to fix my current approach in the near future by making my rover a wall crawler.  This project was a lot of fun!
