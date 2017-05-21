## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1_output.jpg "Road Transformed"
[image3]: ./test_images/threshold.jpg "Binary Example"
[image4]: ./test_images/warp_test.jpg "Warp Example"
[image5]: ./test_images/visual_test.jpg "Fit Visual"
[image6]: ./test_images/draw_test.jpg "Output"
[video1]: ./white.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at No.5 cell in `Udacity-Advanced-Lane-Finding.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in No.3 cell in the file `Udacity-Advanced-Lane-Finding.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[565, 460],
    [265, 700],
    [1125, 700],
    [760, 460]])
dst = np.float32(
    [[240, 0],
    [320, 720],
    [960, 720],
    [1180, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 565, 460      | 240, 0        | 
| 265, 700      | 320, 720      |
| 1125, 700     | 960, 720      |
| 760, 460      | 1180, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

In No.6 cell in `Udacity-Advanced-Lane-Finding.ipynb`, I used the histogram method to detect lane line when we didn't detect good lane line in the last iteration. And in No.7 cell in `Udacity-Advanced-Lane-Finding.ipynb`, I used a method to count all the pixels within a specific margin around the previous good lane line as candidates to determine the lane line. For these pixels, I passed their coordinates into the np.polyfit function to get the 2nd order polynomial coefficients.
![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in get_curve() function in No.10 cell in `Udacity-Advanced-Lane-Finding.ipynb`. I get the average 2nd order polynomial coordinates and passed them to get_curve() function. In this function, it will transfer the pixel space to meter space based on the parameter provided by Udacity, and we fit new coordinates in a new 2nd order polynomial in meter space then we can easy get the curve from using the formula. For the position of the vehicle with repsect to center, I assume that the middle of the image is where the center of the car, so pass the 720*y_meter_per_pixel into the polynomial and get the position of the lane line. Substract these two values then we can get the position of the vehicle with respect to the two lane lines. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in No.8 cell in my code in `Udacity-Advanced-Lane-Finding.ipynb` in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./white.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

First, I simply implemented the method was tought into the course, and in the project video, some frames with shadow or cars near the lane cause the detected lane change their shape. To solve this problem, I first found out it was caused by the car and the edge of the road. So I adjusted the thresholds to remove the affect from unexpected objects, and I also used the average detected lane line pixels to fit the 2nd order polynomial. This also help reduce the affect from unexpected objects. Because the perspective transform is hardcoded, if the car is on a hill or uneven road, the result could be not parallel. Because I was using histogram method to start to detect the lane line, sometimes the edge of the road would be inside the perspective transformed image, this method would consider the edge of the road is the left lane line. To make it more robust, I could search the max two values from middle of the image to two sides of the image and set the position with higher index as search start point.
