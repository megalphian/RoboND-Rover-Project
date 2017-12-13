## Project: Search and Sample Return
### By Megnath Ramesh

---

[//]: # (Image References)

[image1]: ./misc/Rock_test.png
[image2]: ./misc/Filtered_rock.png
[image3]: ./calibration_images/example_rock1.jpg

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here is how I approached each point of the above Rubric to create a solution for each of them

---
### Writeup / README

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

The functions provided in the Jupyter notebook contained the code required to detect the navigable path. The color_thresh function implements a high-pass rgb filter to pass the bright pixels in the image as the terrain. I modified it into a band-pass filter to enable filtering any color band I want. I did this by adding an argument to the function, namely below_rgb_thresh to indicate the upper limit of the rgb band. Using this function, I was able to isolate the obstacles by using the inverse of the spectrum used for the path (From 0,0,0 to 160,160,160). For the rock detection, I experimented with different values around the yellow band and got best results with the range between (50,50,0) and (256,256,38). I was able to get a binary mask showing the pixels of the rock, the obstacles and the navigable path.

##### Test picture for rock detection
![alt text][image1]
##### Binary mask reult
![alt text][image2]
#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.
And another!


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  



![alt text][image3]
