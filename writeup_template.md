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

[image1]: ./examples/calibration1.jpg "Undistorted"
[image2]: ./examples/calibration1corrected.jpg "Road Transformed"
[img3]: ./examples/test1.jpg "distorted road"
[img4]: ./examples/test1corrected.jpg "Undistorted road"
[image3]: ./examples/binaryex.png "Binary Example"
[image4]: ./examples/binaryexTransformed.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/test3.jpg "Output"
[image7]: ./examples/calibration1.jpg "Undistorted"
[video1]: ./examples/project_video.mp4 "Video"
[video2]: ./examples/challenge_video.mp4 "Challenge video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 4th code cell of the IPython notebook located in "./P1.ipynb". The code in this cell is used in code cell #6 to undistort the images.

I start by using getpoints to preparing and obtaining "object points" through the use of  function, which will be the (x, y, z) coordinates of the chessboard corners in every chess board images.
 Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  
 Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.
`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  
I then return both of these points and save it in a points variable for future use.
I then used points to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. 
 I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][img3]
I run the methods above through a loop and adding the above image to the test images folder such that it is undistorted via the calibrated camera.
![alt text][img4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In cell 8 of p1.ipynb through from line 5 to line before perspective_transform( combined_binary). I used the combined binary of sobelx, hue and saturation channel with threshholds of (20,100) for sobelx, (170, 255) for saturation and (50,100) for hue. 
Below is an example image of the a binary
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is implemented in function perspective_transform in cell 4 of p1.ipynb. The `perspective_transform()` function takes as inputs an image (`img`), and assumes source (`src`) and destination (`dst`) points.  
I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[708,462],[1137,720],[189,720],[574,462]])
dst = np.float32([[1137,0],[1137,720],[189,720],[189,0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 708, 462      | 1137,0        | 
| 1137,720      | 1137,720      |
| 189,720       | 189,720       |
| 574, 462      | 189, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This was done in the polylines function. I use a sliding window of 9 along with the calculation of max arg and min arg to find the left and right indices that fall within the range. 
I then fit a second order polynomial within the indices. Did not save a picture here as I've used the draw on undistorted function and removed the previous drawing on the intermediate warped binary.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This was done above the line trackline.radius_of_curvature = left_curverad, right_curverad where left_curverad and right_curverad was calculated.
I do this by fitting a second order polynomial through lefty, leftx and righty, rightx augmented by meters per pixel and then applying the calculation of the curvature in meters 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in p1.ipynb in the function `draw_on_undistorted`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./examples/project_video.mp4)

link to challenge video result ./examples/challenge_video.mp4
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

As I started working with the project video I've noticed a few frames where the pipeline failed to capture the lane lines correctly. I then use the Line class to keep track of the difference
 between the current left polynomial coefficient against last left coefficients  and right coefficients against right coefficients. I added code to allow the code to use the same base while
  the coefficients have not deviated by a great amount. I also added code to specify if both using the last detected base and new base gives bad coefficient then simply draw the last result that worked.
  This seem to work for project video. However moving onto the challenge video it is noted that the pipeline tends to pick up dark parts. Perhaps this has something to do with having hue considered in the pipeline.
  I have tried to mitigate this by adding condition that if anything is detected within 300px of center then try searching again and lastly draw with last good fit. This seems to remove some bad detections but does
   make the video look very weird with oddly shaped curves and still capturing shaded figures on the out side of the lane lines. If I were to continue I would have tried removing hue from the binaries and perhaps strictly enforce a search window.
The last good base was assumed for calculating of center. This is a good estimate but better metrics can be taken such that a more exact running metric of meters from center is tracked.