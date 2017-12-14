## Project: Search and Sample Return
### By Megnath Ramesh

---

[//]: # (Image References)

[image1]: ./misc/Rock_test.png
[image2]: ./misc/Filtered_rock.png
[image3]: ./misc/Simulator_run.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here is how I approached each point of the above Rubric to create a solution for each of them

---
### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

The functions provided in the Jupyter notebook contained the code required to detect the navigable path. The `color_thresh()` function implements a high-pass rgb filter to pass the bright pixels in the image as the terrain. I modified it into a band-pass filter to enable filtering any color band I want. I did this by adding an argument to the function, namely `below_rgb_thresh` to indicate the upper limit of the rgb band. Using this function, I was able to isolate the obstacles by using the inverse of the spectrum used for the path (From 0,0,0 to 160,160,160). For the rock detection, I experimented with different values around the yellow band and got best results with the range between (50,50,0) and (256,256,38). I was able to get a binary mask showing the pixels of the rock, the obstacles and the navigable path.

```
def color_thresh(img, above_rgb_thresh=(160, 160, 160), below_rgb_thresh=(256, 256, 256)):
    # Create an array of zeros same xy size as img, but single channel
    color_select = np.zeros_like(img[:,:,0])
    # Require that each pixel be above all three threshold values in RGB
    # above_thresh will now contain a boolean array with "True"
    # where threshold was met
    thresh = ((img[:,:,0] > above_rgb_thresh[0]) \
                & (img[:,:,1] > above_rgb_thresh[1]) \
                & (img[:,:,2] > above_rgb_thresh[2])) \
                & (img[:,:,0] < below_rgb_thresh[0]) \
                & (img[:,:,1] < below_rgb_thresh[1]) \
                & (img[:,:,2] < below_rgb_thresh[2])
    # Index the array of zeros with the boolean array and set to 1
    color_select[thresh] = 1
    # Return the binary image
    return color_select
```

##### Test picture for rock detection
![alt text][image1]
##### Binary mask reult
![alt text][image2]

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.
The process image should take in the video feed from the robot and map the navigable terrain, obstacles and rock samples in world coordinates.

1) The first step is to do a perspective transform of the camera feed. Using the source and destination algorithm discussed in the class portal, I used the function `perspect_transform` to create the warped images.
```
source = np.float32([[14, 140], [301 ,140],[200, 96], [118, 96]])
destination = np.float32([[img.shape[1]/2 - dst_size, img.shape[0] - bottom_offset],
                [img.shape[1]/2 + dst_size, img.shape[0] - bottom_offset],
                [img.shape[1]/2 + dst_size, img.shape[0] - 2*dst_size - bottom_offset],
                [img.shape[1]/2 - dst_size, img.shape[0] - 2*dst_size - bottom_offset],
                ])
warped = perspect_transform(img, source, destination)
```
2) The next step is to create the binary masks for the path, rock and obstacles using the new color_thresh function
```
path_threshed = color_thresh(warped)
rock_threshed = color_thresh(warped, (50, 50, 0), (256,256,38))
obstacle_threshed = color_thresh(warped, (0,0,0), (160,160, 160))
```
3) Convert the thresholded pixels to rover centric coordinates using the provided function
```
 x, y = rover_coords(path_threshed)
 r_x, r_y = rover_coords(rock_threshed)
 o_x, o_y = rover_coords(obstacle_threshed)
```
4) Convert the rover centric coordinates into world coordinates using the x,y,yaw values from the csv file.
```
 w_x, w_y = pix_to_world(x, y, data.xpos[data.count], data.ypos[data.count], data.yaw[data.count], data.worldmap.shape[0], 10)
 r_w_x, r_w_y = pix_to_world(r_x, r_y, data.xpos[data.count], data.ypos[data.count], data.yaw[data.count], data.worldmap.shape[0], 10)
 o_w_x, o_w_y = pix_to_world(o_x, o_y, data.xpos[data.count], data.ypos[data.count], data.yaw[data.count], data.worldmap.shape[0], 10)
```
5) The worldmap is an image with 3 different rgb channels. Based on the filtered binary masks, the areas occupied by each feature are set to each channel of the worldmap as follows. Each value is set to 255 to compensate for the high speed of the video.
```
 data.worldmap[o_w_y, o_w_x, 0] = 255
 data.worldmap[r_w_y, r_w_x, 1] = 255
 data.worldmap[w_y, w_x, 2] = 255
```


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

After finishing the notebook, I had the same procedure of processing the image in mind when filling out the `perception_step()` function. I first modified the `color_thresh()` function to accept a color band as an input. After that, I replicated the functionality of the `process_image()` function in `perception_step()`, with two additions.

- To create the vision image on the left side of the screen, I used the binary masked images and amplified the RGB values to clearly visualize the rover's point of view.

- At the end of the function, I recorded the rover's displacement from it's origin and it's angle from the origin by calculating the polar coordinates of its position. This is then updated for the next processing run.

Upon testing with the original decision matrix in `decision.py`, I was able to reach the required 40% mapping at above 60% fidelity. There were some edge cases which I will discuss in the later part of this report. As it stands right now, no changes were made in `decision.py`

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

The simulator was launched with the resolution of 1280x800 at a Good graphics quality setting. The rover was able to map the environment, locate the rock samples and obstacles and pass the minimum requirements ~70% of the time with no manual intervention. Below is a snapshot of one such run passing the necessary requirements.

![alt text][image3]

##### Inferences:

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
