## Report

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./report_img/calibration.jpg "camera calibration"
[image2]: ./report_img/undistort.jpg "undist"
[image3]: ./report_img/color_grad.jpg "color_grad"
[image4]: ./report_img/perspective.jpg "perspective"
[image5]: ./report_img/histogram.jpg "histogram"
[image6]: ./report_img/sliding_windows.jpg "sliding_windows"
[image7]: ./report_img/fit_frame.jpg "fit_frame"
[image8]: ./report_img/curve_eq.jpg "curve_eq"
[image9]: ./report_img/draw.jpg "draw"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd, 4th code cell of the IPython notebook located in "./Advanced_Lane_Finding.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at 7th code cell of the IPython).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is at 8th code cell of the IPython. 

I defined an area with np.array() and use cv2.warpPerspective to perform Birds-eye view - image for the area
The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[(490,470), 
                   (img.shape[1]-490,470),
                   (img.shape[1], img.shape[0]*0.9), 
                   (0, img.shape[0]*0.9)]])
                   
dst = np.float32([[0, 0], 
                  [img.shape[1], 0],
                  [img.shape[1], img.shape[0]], 
                  [0, img.shape[0]]])

```
This resulted in the following source and destination points:


| Source                | Destination   | 
|:---------------------:|:-------------:| 
| 490, 470              | 0, 0          | 
| Width-490, 470        | Width, 0      |
| Width, Height x (0.9) | Width, Height |
| 0, Height x (0.9)     | 0, Height     |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for my perspective transform is at 9th~13th code cell of the IPython.

I use histogram and sliding windows on the binary bird-eye image to identify the lane lines and fit the 2nd order polynomial. And then, I use the result from sliding windows as reference to fit the 2nd order polynomial for the next frame

using histogram to identify most detected binary pixels = 1 on x axis
![alt text][image5]

applied the result from histogram to detect lane lines
![alt text][image6]

used the result from sliding lines as reference on next frame
![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for my perspective transform is at 14th~15th code cell of the IPython.

Curvature formula is:
![alt text][image8]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for my perspective transform is at 16th code cell of the IPython.
I applied the result from step 4 and use inverse matrix got from 3 to transform back from bird-eye image to original image.

Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. In the challenge case, when the car near the bridge, my pipeline is hard to identify the lane lines
2. In the challenge case, when there's dark lines on the ground, it will make the line detection wrong.

I used the Line() class to save previous fit data and also sanity check to make sure the left and right lines are good. When the line detect is failed, I can use the data from previous result. But it still need more improved.

