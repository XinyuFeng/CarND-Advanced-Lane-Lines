## Writeup Template

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

[image1]: ./ref_img/calibrate.png "Calibrate"
[image2]: ./ref_img/undistort.png "Road Transformed"
[image3]: ./ref_img/combined_binary.png "Binary Example"
[image4]: ./ref_img/transform.png "Warp Example"
[image5]: ./ref_img/fit_polynomial.png "Fit Visual"
[image6]: ./ref_img/fit_polynomial.png "Output"
[video1]: ./result.mp4 "Video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./Advance_Lane_Find.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. I successfully detected chesboard corners and obtained this result:

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of saturation and gradient thresholds to generate a binary image (thresholding steps at cell 6 in `./Advance_Lane_Find.ipynb`).  Firstly, I test different threshold of saturation and find out that[180, 255] can perferm well while lowering the threshold can cause more ireelevant points to be added in, and increasing it can make the line more blur. Then, to make thing more precise, I added sobel gradient on the direction of X and choose threshold [60, 100].Finaaly, I combined these restrictions together and out put a binary image. Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspec_trans()`, which appears in the 5th code cell of the IPython notebook.  The `pespec_trans()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[480, 480], [800, 480], [1250, 720], [0, 720]])
dst = np.float32([[0, 0], [1250, 0], [1250, 720], [0, 720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 480, 480      | 0, 0        | 
| 800, 480      | 1250, 0      |
| 1250, 720     | 1250, 720      |
| 0, 720      | 0, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear nearly parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I used classical window search method to identified lane-line pixels. First, I identified the two bottom points to get started. I choose 10 windows per lane. so I calculate the center of those windows level by level based on those points laying in window. Next, I concatenate those points that belongs to the same lane, and call polyfit() function in opencv, to get a second order curves with coefficients returned, that can fit those points to a real curved lane.
The result is like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 9th cell (85-93 lines) of my notebook. I translated the measurable units from pixel to meters, and then, re-polyfit from translated points. And finally, I calculated it using equation for radius of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 9th cell (105-118 lines) of my notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
For each image frame of thee video:
1. I make necessary preprocess: calibrate, undistort, perspective transform and taking threshold.
2. If this's the first time I search lanes, or the last lane detection was a bad detect, I will use sliding window method to find a 2nd order curve that fit both left and right lanes. If I've already detected lanes, I just use the previous fit and get those lane points in +- margins areas from the curve, then fit a new lane from those new points.
3. To validate this fit, I check it with my last fitted lane. If the two lanes' difference is huge, means the new fit is a bad fit, so I just abort it. If the difference is small, I just taking average of the 6 ost recent fitted lanes to be the best fit, so as to smoothing the result.
4. I calculate curvature and track car's position as well.

Problems:
The image processing is not thorough enough, since I found out that when the car is in some darken area, or there are some irrelevant black lanes on the road, the combined binary image can also include those lanes, and our interested yellow lanes will be blur due to the dark. This can lead to bad polyfit (I noticed in challenge video). I think the dark area can not be avoid. To improve it, I can either use some other color channels to block those black lanes, or increasing the color contrast to augment image brightness. In my project, I just ignore the bad fit.
