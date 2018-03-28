# Advanced Lane Finding Project

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

[image_cal_undistort]: ./images/cal_undistort.png "Calibration Image Undistorted"
[image_undistort]: ./images/undistort.png "Undistorted Example"
[image_binary]: ./images/binary.png "Binary Example"
[image_warp]: ./images/warp.png "Warp Example"
[image_histogram]: ./images/histogram.png "Histogram Example"
[image_sliding_window]: ./images/sliding_window.png "Histogram Example"
[image_final]: ./images/final.png "Final Output"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

*code cell #3 and #4 in `Advanced Lane Finding.ipynb`*

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `object_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `image_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `object_points` and `image_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![Undistorted Image][image_cal_undistort]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

*code cell #5 in `Advanced Lane Finding.ipynb`*

Applying the same undistort on the test image will produce the following result:
![alt text][image_undistort]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

*code cell #6 in `Advanced Lane Finding.ipynb`*

I used a combination of color and gradient thresholds to generate a binary image. Here's an example of my output for this step.

![alt text][image_binary]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

*code cell #7 in `Advanced Lane Finding.ipynb`*

The code for my perspective transform includes a function called `warp()`. The `warp()` function takes as inputs an image (`img`). I chose the hardcode the source and destination points as following:

| Source        | Destination   |
|:-------------:|:-------------:|
| 595, 440      | 250, 0        |
| 690, 450      | 1030, 0       |
| 1040, 670     | 1030, 720     |
| 270, 670      | 250, 720      |

Here is the result:

![alt text][image_warp]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

*code cell #8 and #10 in `Advanced Lane Finding.ipynb`*

Then I take a histogram on all pixels in the lower half of the image and find the peaks as starting points for finding the lines.

![alt text][image_histogram]

And then I use sliding window from the starting points to find and follow the lines up to the top of the image. And then I stored all the points within the sliding windows and fit them with a 2nd order polynomial.

![alt text][image_sliding_window]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

*code cell #11 in `Advanced Lane Finding.ipynb`*

I first converted the 2 detected lines from pixel space to world space by multiplying them with corresponding meter per pixel ratios and then fit a 2nd order polynomial using world space values. Then I use the radius of curvature formula to calculate 2 radius of curvature values and average them to get the radius of curvature of the lane.

For the position of the vehicle with respect to center, I calculate the position of the 2 lines at the bottom of the image using the 2 polynomials. Then subtract the half of the image width from the 2 points. Left line will have a negative position (meaning on the left side of the lane center) and right line will have a positive position (meaning on the right side of the lane center). Lastly, I take the average of the 2 line positions and  multiply by the horizontal meter per pixel ratio to get the position of the vehicle with respect to the lane center in centimeter.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

*code cell #13 in `Advanced Lane Finding.ipynb`*

I implemented `draw_lane()` in this step and here is an example of my result on a test image:

![alt text][image_final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

For searching of lines from the video, instead of starting from scratch using sliding windows for every frame, I used the polynomials of the lines from the previous frame. This approach could speed up the search process. However, it may cause problem when there are too many noise in the binary warped image, especially at the end of the second bridge with the color difference between the bridge and the road and shadow of the trees. This affected the lane detection missed seriously. To avoid this, I compare the bottom position of both the left and right lines with the position from the previous frame. If the position has changed too much (more than 10%), I start the sliding window search again.
