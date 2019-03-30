## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./output/color_threshed.jpg
[image2]: ./output/obstacle_threshed.jpg
[image3]: ./output/rock_threshed.jpg
[image4]: ./output/test_mapping.jpg
[image5]: ./output/train_mapping.jpg
[image6]: ./output/autonomous_mode.png
[image7]: ./output/pickup_rock.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
I added two functions other than `color_thresh()`. Those functions are `obstacle_thresh()` and `rock_thresh()`. In obtaining the threshold range for each function, I found it tedious to use the interactive window of matplotlib. Instead I opened the images in matlab and used the color thresholder to obtain the color thresholds. Then transfered the color ranges to match the python code.  
For the `rock_thresh()` function, I transfered the RGB image to HSV and then performed the color threshold. 

![color_thresh()][image1]
![obstacle_thresh()][image2]
![rock_thresh()][image3]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
1\) I defined the source points by using the interactive window of matplotlib and simply used the destination points from `perspective transform`.  
2\) I used the predefined `perspect_transform()` function to apply perspective transform and obtain the warped image.  
3\) I ran the warped image through my three color threshold functions and obtain three seperate thresholded images.  
4\) I ran each of the thresholded images through `rover_coords()` to convert the pixel values to rover-centric coordinates.  
5\) Finally, I set the world size to 200x200 pixels and set the scale factor to 10 since the ground truth image has a resolution of 1m<sup>2</sup>/pixel and the rover-centric coordinates have a resolution of 0.1m<sup>2</sup>/pixel.  
6\) When updating the worldmap, I set the navigable area to blue, obstacle area to red and the rock area to green. However the rock area usually overlaps with the obstacle area. Thus rocks are usually shown as yellow. Also in order to prevent the newly found navigable area from overlapping with the previously found obstacle areas, I set the overlapping obstacle areas to 0.

![test_mapping.jpg][image4]
![train_mapping.jpg][image5]

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
Mostly the functions from the jupyter notebook were copy and pasted into the `perception.py` file. Some modifications were made regarding the variable names. For instance, variables such as `img` and `data` were modified to be associated with the `Rover` object. Also, I chose to only update the worldmap when the pitch and roll values were less than 300. This is because I found that when the values were greater than 300, the rover was making hard turns and thus the map was being distorted.  
For `distortion.py`, I added code for an emergency maneuver when the rover is stuck and also code for locating and picking up rocks. When the rover is stuck for more than 5 seconds I tell the rover to rotate 45 degrees. When a rock is located I tell the rover to steer towards the rock and stop near it. When the rover is near enough the original code will pick it up.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

I took the basic approach of steering the rover in the direction of most navigable pixels. This was effective and I was able to map most of the terrain. However the rover sometimes continued to circle a particular part of the map. It would better if I used a `follow the wall` approach. I used a very tight color threshold in order to increase fidelity. The downside of doing so was that the rover sometimes could not distinguish the sky from the shadows and some parts of the map were unreachable. I would be able to obtain a better result if I spent more time experimenting on the threshold values. I found rocks by starting to hit the brakes at 1m from the rocks. I did not conisider the velocity. Therefore sometimes the rover would charge at the rocks or stop too short of the rocks. Since most rocks are located near a wall, charging would still put the rover very close to the rocks and the rover is able to pick up rocks even when it is standing on them. I took care of the stopping too short with an additional if statement. Putting velocity into account would make a much smoother pickup.

![autonomous_mode.png][image6]
![pickup_rock.png][image7]



