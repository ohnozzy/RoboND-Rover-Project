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

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[image4]: ./thresh.png
[video]: ./video.jpg
[mask]: ./mask.png
[result]: ./autoresult.png


## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

The `color_thresh` function takes 3 arguments, `img`, `lower` and `upper`.

It first converts img from BGR color space to HSV color space using the `cv2.cvtColor` function. Then it use `cv2.inRange` to mark the pixels fall between `lower` and `upper`.

Since the rocks are yellow, it is easier to specify the color range for yellow in HSV space than BGR space.

The image below shows the result. The upper left image is the camera input after perspective transformation. The upper right one is the navigable area identified by `color_thresh` function. The bottom left is the result of overlaying the upper right image on top of the upper left image. The bottom right one shows the rock location identified by `color_thresh`

 
![color_thresh result][image4]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

It first use `perspect_transform` to transform the camera image to topdown view. Then it use `color_thresh` with 3 color ranges defined in `path_color`,`rock_color` and `obstacle_color` to detect navigable area, rocks and obstacles.

Since some areas are always dark after `perspect_transform` and the distance and angle to the objects far away cannot be accurately measured. The mask below is applied to the detection result. (idealy it should be a circular sector.)
![mask][mask] 

Then the results are translated to rover coordinates by `rover_coords` and then to world coordinates by `pix_to_world`. Then the 3 channels of `worldmap` are updated with the world coordinates of the navigable area, rocks and obstacles. Each object occupies one channel. Whenever an object is detected, the corresponding channel of the pixel on the worldmap is incremented by 1. A pixel is classified as navigable area, rock or obstacle base on the channel with the highest count.

![process_image][video]
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

The `perception_step()` is nearly the same as `process_image()` except that the rover coordinates of the navigable area is also transformed to polar coordinate and then assigned to `Rover.nav_dists` and `Rover.nav_angles`.

The support function `create_output_images(Rover)` is modified.

```python
plotmap[:, :, 2] = navigable
```

is changed to
 
```python
plotmap[:, :, 2] = 255 * likely_nav
```
So that the location is considered as navigable only when `navigable > obstacle`.

No change is made to `decision_step()`.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

##### Graphics Parameters
* Screen Resolution: `640 X 480`
* Graphics Quality: Fast
* FPS: 16

##### Result
* Mapped: 40~60%
* Fidelity: 90% 
* located rocks: >1

![result][result]

Fidelity is about 90%. However, Mapped area fluctuates quite a lot depending on the initial pose. The `decision_step` often get trapped into circle instead of exploring the whole map. 

##### Improvement

In order to fully explore the map, the `decision_step` should use a path finding algorithm like Dijkstra's algorithm to find a path from the current location to an unmapped location and then follow the path. 



