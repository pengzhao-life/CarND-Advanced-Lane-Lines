## Writeup 

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

Camera Calibrtion
1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

Pipeline (Single Image)
1. Apply a distortion correction to raw images.
2. Use color transforms, gradients, etc., to create a thresholded binary image.
3. Apply a perspective transform to rectify binary image ("birds-eye view").
4. Detect lane pixels and fit to find the lane boundary.
5. Determine the curvature of the lane and vehicle position with respect to center.
6. Warp the detected lane boundaries back onto the original image.
7. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Pipeline (Video)

[//]: # (Image References)

[image1]: ./output_images/calibration1_undistort.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/test1_binary.png "Binary Example"
[image4]: ./output_images/test1_warped.png "Warp Example"
[image5]: ./output_images/test1_masked_warped.png "Masked Warp Example"
[image6]: ./output_images/test1_lane.png "Fit Visual"
[image7]: ./output_images/test1_final_img.png "Output"
[video1]: ./output_videos/project_video.mp4 "Video"

#### All the code is in the IPython notebook located in `./P2.ipynb`


### Camera Calibration

#### 1. Briefly state how I computed the camera matrix and distortion coefficients. Provide an output of a distortion corrected calibration image.

The code for this step has two major functions `compute_calibration_params()` and `apply_undistort()`, which are in the 3rd and 5th code cell of the 'Camera Calibration' section of the IPython notebook. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image `camera_cal/calibration1.jpg` using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I apply the function `apply_undistort()` (in code cell 5 as mentioned above), which provides correction matrix and distortion coefficients, to the test image `camera_cal/calibration1.jpg`:
![alt text][image2]

#### 2. Describe how (and identify where in my code) I used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (i.e. saturation) and gradient thresholds to generate a binary image. The code is at 8th code cell in the `P2.ipynb`.  Here's the binary image of my output for the test image 1:

![alt text][image3]

#### 3. Describe how (and identify where in my code) I performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in 10th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto the same test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

I noticed that there are some noise on the left and right boundary area, and this makes poly fitting less accurate. I apply a mask to mask them out. The code is 12th code cell function `mask_binaryimg()`. The masked result is like this:

![alt text][image5]

#### 4. Describe how (and identify where in my code) I identified lane-line pixels and fit their positions with a polynomial?

The code is in 13th code cell, where the function `find_lane_pixels()` is to detect the pixels in a sliding window, and function `fit_polynomial()` does a 2nd order polynomial fitting:

![alt text][image6]

#### 5. Describe how (and identify where in my code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code is in 14th code cell, where the function `measure_curvature_real()` is to calculate the curvature of polynomial functions and the vehicle position in meters.

The curvature equation as below is calculated for x and y respectively:

Rcurve = ((1 + (2* A * y * ym_per_pix + B)**2)**1.5) / 2|A|

I cacluate the left land and right lane, get the lane center from them, then subtract the image center. Make sure to convert from pixels to meters.

left_lane_bottom = (left_fit[0]*y_eval)**2 + left_fit[0]*y_eval + left_fit[2]

right_lane_bottom = (right_fit[0]*y_eval)**2 + right_fit[0]*y_eval + right_fit[2]

position = ((left_lane_bottom + right_lane_bottom)/2. - 640) * xm_per_pix

#### 6. Provide an example image of my result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 15th code cell in my code in the function `warp_lanes_on_img()`.  Here is an example of my result on the test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to my final video output.  My pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The pipeline for video is almost the same as the pipeline for single image, except that the fitting parameters from the previous frame can be used as starting point for the current frame. So the function for fitting is changed to `search_around_poly()` in 20th code cell.

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues I faced in my implementation of this project.  Where will my pipeline likely fail?  What could I do to make it more robust?

The approach I took takes a few steps: distortion correction, thresholding from undisted image to binary image, perspective transform to bird eye view, 2nd order polynomial fitting , then perspective transform back to the front view image.

This pipeline works well for the good quality image/video. For a more robust system, what I can think of is to:

* Apply preprocessing to get a better quality image
* Improve the thresholding method for a better binary image. This will make poly fitting more accurate
* Modify the current fitting method (i.e. the search around method), check the fitting result, if not good (how to measure this?), don't use the fitting parameter from previous and do a clean search like the single image does.

I live in Boston. The lane may not be visible in the winter, since the lane may be covered by snow or ice. As a human, I guess where the lane is by the width of the road and other cars position. I wonder how algorithms do that.
