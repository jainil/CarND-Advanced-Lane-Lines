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

[image1]: ./output_images/undistorted.png "Undistorted Chessboard"
[image2]: ./output_images/test_undistorted.png "Road Transformed"
[image3]: ./output_images/binary_output.jpg "Thresholded binary image"
[image4]: ./output_images/binary_warped.jpg "Perspective transformed binary image"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./advandced_lanes.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I applied the distortion correction with the correction matrix derived in the previous step. The results are as follows:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (fourth code cell in `advandced_lanes.ipynb`). I used the threshold values from the trials done in the lessons.  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform appears in the 3rd code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points with the following mapping:

```
[ 580.  450.] => [ 0.  0.]
[ 705.  450.] => [ 1280.     0.]
[ 1280.   720.] => [ 1280.   720.]
[  60.  720.] => [   0.  720.]
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the histgram based lane detection(code cell 7) to detect the lane pixels and then fitted them to a second order polynomial, as seen below:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implmented this in code cell 8.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 9.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach I took for this project was fairly along the lines discussed in the tutorial sessions. The pipeline as it stands has a few brittle points(all made painfuly evident by the performance of this particular implementation on the challenge video):
1. The determination of the perspective transform: The computation of the perspective transform matrix involves a manual predetermination step where I picked the source and destination points for the transform. This is very approximate measure and quite specific to the particular video frames under consideration.
2. The parameters for thresholding the various gradients were also tuned by me to work for the test images. They may not be robust under conditions where the scene's colors and contrasts are drastically different.
3. After application of the thresholding functions there may be too few (or none) points to fit an accurate polynomial for the lane lines.

For points 1 and 3 above, I think one solution may be to initialize the detector with a conception of what straight lanes should look like given this camera and its mounting position on the car. This can then be used to create the source and destination point pairs for the perspective transform. Additionaly, this can be used as the fill-in for frames where the lane detection does not work well.
For addressing point 2, we can make an adaptive system that tries a few different parameters and combinations of operators every once in a while to see what results in the clearest fit to lane lines. Also perhaps there is a better combination of parameters to try out than those I could test out for this project.
