

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

[image1]: ./output_images/CameraCalibration.png "Camera Calibration"
[image2]: ./output_images/UndistortTest.png "Camera Calibration"
[image3]: ./output_images/ExploreThresholds.png "Threshold Exploration"
[image4]: ./output_images/thresholds.jpg "Applied Thresholds"
[image5]: ./output_images/PerspectiveTransform.png "Perspective Example"
[image6]: ./output_images/LanesPathOverlaid.png "Lanes and Path Overlaid"
[image7]: ./output_images/ROC.png "Radium Of Curvature"
[image8]: ./output_images/LaneMapped.png "Mapped Lane"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1.   [Here](https://github.com/gvogety/CarND-AdvancedLaneFinding/blob/master/README.md) is the writeup for this project.  

You're also reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Assuming chessboard is fixed in (x,y) plane, objpoints are the same for each image.  Depending on the size of the chessboard, all corners may not be detected successfully. Whenever corners are detected, objpoints is updated with objp and "imagepoints" is updated with (x,y) pixel position of each corner detected.  

`objpoints` and `imgpoints` are used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is the undistorted test image after camera is calibrated with the chessboard images.

![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In the `Thresholds` section of the accompanying IPython notebook, starting around code cell#5, various threshold utility functions are defined. I created various visualtization python functions to select optimal thresholds. Using `interact` facility from `ipywidgets`, I explored various color tranforms, gradients with appropriate thresholds.

 Here's an example of my output for one of the exploration steps. 

![alt text][image3]

After this thorough exploration, Sobel X & Y thresholds, Magnitude and Direction gradients and s-channel are used for final image. All thresholds are coded in 'apply_thresholds()' python function.

Here is an example of an image with all various gradients and thresholds explored. Code for this is in the 'apply_thresholds()' python function.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes couple of function called `calculate_perspective_matrices()` and `perspective_transform()`, which appears in *Perspective Transform* section of the iPython Notebook.  The first function calculates the transform matrices using following coordinates.  The `perspective_transform` function takes as inputs an image (`img`), and used the matrices calculated from the previous function to give transformed images.

```python
	src = np.float32([[580,460],[710,460],[150,720], [1150,720]])    
    dst = np.float32([[450,0],[1280-450,0],[450,720],[1280-450,720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 450, 0        | 
| 710, 460      | 830, 0      |
| 150, 720     | 450, 720      |
| 1150, 720      | 830,720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. Following is such an example.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

*Lane Finding* section of the accompnying IPython notebook has the various functions to find lanes. 

After the gradients and thresholding functions have found a perspective transfomed image, Histogram of the pixels is used to find the lanes. Location of the largest pixel density is used as the potential lane locations. 

The images is divided into a series of windows along the y-asis and the windows are analyzed along the lane locations found above. These windows have a margin around the lane location. As each side of the picture is scanned for lanes, potential pixels that form the lanes are collected together.

Left and Right lane polynomials are found by using `np.polyfit`. These polynomials are used to find the precise lane on the left and right sides respectively.

Using the lanes found using the polynomials, the lanes are drawn on the image and reverse persepctive transform is done on the resulting pixel locations to project a path onto the original image.

Following image shows the resulting image with a test image.

![alt text][image6]

`Note:` I did not try to optimize lane detection by using lane locations from the previous frames and reset to recalculating the lanes after sanity checks failed. This is an area of optimizations I intend to implement during second term. (I lost 2 months in Term1 due to life, :-| )

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of Curvature is calculated in the python function `radius_of_curvature_and_center_distance()` using the approach from the lesson. pixel to meter conversion is adjusted based on the lane widths that I observed in various perspective transforms of the test images. These can be a a little bit off from the ideal ones.

Here is an image of the radius of curvature on one of the test images.

![alt text][image7]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is implemented in the python function `project_path()`. Inverse matrix from the `calculate_perspective_matrices()` function is used to project the lanes/polygons found on perpective transformed image on to the original image. Here is one such example.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
