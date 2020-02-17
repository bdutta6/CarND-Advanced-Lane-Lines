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
[image00]: ./camera_cal/calibration1.jpg "Corners marked calibration image"
[image01]: ./corners_marked_images/calibration1.jpg "Corners marked calibration image"
[image10]: ./undistorted_calibration_images/calibration1.jpg "Undistorted calibration image"
[image1]: ./test_images/straight_lines2.jpg "Raw"
[image2]: ./calibrated_images/straight_lines2.jpg "Undistorted"
[image3]: ./binary_images/straight_lines2.jpg "Binary Example"
[image4]: ./warped_images/straight_lines2.jpg "Warped binary Example"
[image5]: ./warped_raw_images/straight_lines2.jpg "Warped raw example"
[image6]: ./lane_fitted_images/straight_lines2.png "Warped_fitted_window"
[image7]: ./prev_lane_fitted_images/straight_lines2.png "Warped_fitted_prev"
[image8]: ./unwarped_lane_fitted_images/straight_lines2.jpg "Unwarped lane fitted image"
[video1]: ./project_video_output.mp4 "Video"
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

Firstly, I divide my code into two sections. The first section defines all the functions required to finish the pipeline in various cells. There are various places in this section where I test the functions on the test images. (details below). Once I ensure all the steps function properly on the images, I write the second section of the pipeline which then, runs the algorithm on a video. 

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

My readme file and my IPython notebook code is contained in the following location respectively:
 * "./AdvancedLaneFinding.ipynb"
 * "./Readme.md"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the IPython notebook 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. This is contained in code cell 2. An example of a calibration image with and without the corners overlaid is shown below:

![alt text][image00]

Corner marked image:

![alt text][image01]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()`. This is contained in code cell 3 by defining a `cal_undistort()` function that takes in images and `objpoints` and `imgpoints` and returns the undistorted image with the camera calibration parameters that can be reused through the rest of the code assuming same camera is used for all the images and videos. An example of an undistorted calibrated image is shown below:

Undistorted calibration image:

![alt text][image10]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the calibration measurements above, I now apply the distortion correction to the test images. An example of a distorted and undistorted road image is shown below:

Distorted road image:

![alt text][image1]

Undistorted road image:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (this step is contained in the code cell 4 under the function `binary_transformation()` ).  Here's an example of my output for this step.

Binary Image:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_image()`, which appears in the 6th code cell of the IPython notebook).  The `warp_image()` function takes as inputs an image (`img`), and specifies hardcoded source (`src`) and destination (`dst`) points.  I chose the hardcoded source points by estimating a polygon over the straight portion of the lanes in one of the raw straight images by using a hovering tool. This gives me the following values:


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 212, 700      | 350, 720        | 
| 1090, 700      | 950, 720      |
| 737, 480     | 950, 0      |
| 550, 480      | 350, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Test Image:

![alt text][image1]

Warped Raw Image:

![alt text][image5]

Warped Binary Image:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then, I implemented a histogram analysis approach to find peaks in the left and right portion of the images which denotes the left and right lanes respectively. This analysis is conducted on horizontal window strips in the image. Once the peaks are found, a region of interest is defined around the peaks to extract the lane pixels for fitting a line. 

I then fit my lane lines with a 2nd order polynomial. The codes for this section is contained in code cell 8 which uses the following functions `find_lane_pixels` and `fit_polynomial` functions. This process is depicted in the image below:

![alt text][image6] 

Repeating the above step for each video frame is redundant. So, for subsequent frames, the previous fitted line is used to define a window around it for the entire image using the  `search_around_poly` function . An example of this is shown below:

![alt text][image7]



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The functions used for calculating the radius of curvature can be found in code cell 10. The math for calculating radius of curvature for a curve is well explained in the following link
[link to curvature tutorial](https://www.intmath.com/applications-differentiation/8-radius-curvature.php)

The position of the vehicle with respect to center can be found by calculating the difference between the center of the observed lane and the center of entire image.  
#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines by defining a function `unwarp_image()` in code cell 11.  Here is an example of my result on a test image:

Unwarped Lane marked image:
![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

All the above functions were applied on the video image frames and a lane class was created that stored lane credentials. If previous lanes were detected and were present, then subsequent frames only looked around those fitted lanes to find new lane points.

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This pipeline works well for the project_video. However, it fails for the challenge videos. This is obvious as there are still a couple of parameters that are manually tuned like the binary and edge thresholds. These thresholds are also linked with the lighting conditions. Some dynamic thresholding should help with this. Secondly, approximating the src points for warping and unwarping should also be automatic based on some well defined lane dimensions. 


