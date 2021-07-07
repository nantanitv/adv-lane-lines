# **Advanced Lane Finding Project**

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

[image1]: ./output_images/chessboard_undist/cb_undist1.jpg "Undistorted"
[image2]: ./output_images/undist.jpg "Road Transformed"
[image3]: ./output_images/thresholded_binary.jpg "Binary Example"
[image4]: ./output_images/warped_points.jpg "Warp Example"
[image5]: ./output_images/poly_windows.jpg "Fit Visual"
[image6]: ./output_images/result_with_text.jpg "Output"
[video1]: ./output_videos/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the "Camera Calibration" section of the IPython notebook located in `./P2.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (HSV's V-channel) and x-gradient thresholds to generate a binary image (thresholding steps in the section "Color and Gradient Thresholding").  The thresholding levels I found to work best are (225, 255) for the V-channel, and (30, 100) for the x-gradient.  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is under the "Perspective Transform" section of my notebook.  First, I defined the source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
vertices = np.array([[
    (230,700), 
    (590,450), 
    (690,450), 
    (1080,700)]], dtype=np.int32)

margin = 300

src = np.float32(vertices)

dst = np.float32([
    (margin, undist.shape[0]), 
    (margin, 0), 
    (undist.shape[1]-margin, 0), 
    (undist.shape[1]-margin, undist.shape[0])])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 230, 700      | 300, 720      | 
| 590, 450      | 300, 0        |
| 690, 450      | 980, 0        |
| 1080, 700     | 980, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

I finally applied the perspective transform to the binary image, to be further processed in the succeeding steps.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Before finding the lane pixels, the function `hist()` will help us construct the histogram for the sum of pixels on the bottom half of the input image `img`.  This will be useful in the next step.

To identify the lane pixels, I have included the functions `find_lane_pixels()` and `fit_polynomial()`.

The function `find_lane_pixels()` takes a binary image as an input and map sliding windows over the lane pixels.  The image with the sliding windows `out_img` is returned together with the arrays of x and y lane pixels of the left and right lanes `leftx`, `lefty`, `rightx`, and `righty`.

The other function `fit_polynomial()` takes a binary image `binary_warped` as an input, and returns the processed image `out_img`, the polynomial coefficients of the left `left_fit` and right lanes `right_fit`, and the domain of the polynomial `ploty`.  The function puts the image through `find_lane_pixels()` to first identify the lane pixels, and fit quadratic polynomials on each detected lane line.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Under the section "Measure the curvature of the lane and the vehicle distance from center", I covered the said process with the functions `measure_curvature()` and `measure_offcenter()`.

`measure_curvature()` takes in the domain of the polynomial `ploty`, and the polynomial coefficients of the left and right lanes `left_fit_cr`, `right_fit_cr`.  Then we define the meter-per-pixel conversion ratio in the x and y directions, which was done by visual inspection and referring to the US Highway Codes. The radii of curvature was calculate from the formula and returned for both sides.

`measure_offcenter()` receives the image's shape and the x values in both lanes as inputs.  It takes the bottommost pixel on both sides, find their midpoint, and compare the position to the image's midpoint.  It finally returns the value of position off the center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the section "Warp back to original view" and .  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline works fine under optimal lighting and pavement conditions, but will fail under strong lighting and lanes with split pavements.  This could be improved if we keep track of the previous best-fit polynomials and average it over the course in order to avoid unwanted sudden jumps.
