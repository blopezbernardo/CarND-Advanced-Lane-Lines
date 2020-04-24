## Writeup P2 - Advance Line Finding - B Lopez


[//]: # (Image References)

[image1]: ./output_images/Undist_calibration1.jpg "Undistorted calibration1"
[image2]: ./output_images/Undist_test2.jpg "Undistorted test2"
[image3]: ./output_images/Binary_test2.jpg "Binary Threshold test2"
[image4]: ./output_images/Warped_test2.jpg "Warped test2"
[image5]: ./output_images/Polynomial1_test2.jpg "Lane pixel windows test2"
[image6]: ./output_images/Polynomial2_test2.jpg "Lane Polynomial test2"
[image7]: ./output_images/HighlightLane_test2.jpg "Highlight Lane test2"

[video1]: ./project_video_Processed.mp4 "Video"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `./P2 - Advance Line Finding - B Lopez.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at code cell under heading `2.2. Creation of the Thresholded binary image`). Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform can be found under the heading `3 Perspective transformation`. 
I created the transform matrix with the following source (`src`) and destination (`dst`) points.

```python
src = np.float32(
    [[(img_size[0] / 2) - 45, img_size[1] / 2 + 90],
    [((img_size[0] / 6) - 8), img_size[1]],
    [(img_size[0] * 5 / 6) + 34, img_size[1]],
    [(img_size[0] / 2 + 45), img_size[1] / 2 + 90]])

dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595,     450  | 320, 0        | 
| 205.33,  720  | 320, 720      |
| 1100.66, 720  | 960, 720      |
| 685,     450  | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
https://view5639f7e7.udacity-student-workspaces.com/edit/CarND-Advanced-Lane-Lines/writeup_P2_Blanca_Lopez.md#
![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial.

Under the heading `4 Identify lane-line pixels and fit their positions with a polynomial` is all the code where I fit my lanes to a 2nd order polynomial. 
First I identified the lane pixels:
I did a histogram to the bottom half of the image to identify where the lanes are located in the image.
I split the histogram for the two lines: find the peak of the left and right halves of the histogram.
I set up windows and window hyperparameters, to then Iterate through nwindows to track curvature.
The following image is an example of the lane pixel windows for the image `test2`.
![alt text][image5]

After I have found all our pixels belonging to each line through the sliding window method, I implement the function for the polynomial using np.polyfit. 

The following image is an example of the fitted polynomial for the lanes in image `test2`
![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Under the heading `5 Calculate the radius of curvature of the lane and the position of the vehicle with respect to center` is all the code where I calculated the radius of curvature of the lane and the position of the vehicle with respect to center. 

First I did the conversions in x and y from pixels space to meters.
The radius can be calculated with the following formula:
R = ((1+(2*Ay + B)^2))^(3/2))/(|2A|)
Being A & B the coefficients of the 2nd order polynomial y=Ax^2+Bx+C

The deviation from center line (estimated) with 
ctr=abs((((rightx_base-leftx_base)/2)+leftx_base-midpoint)*xm_per_pix) 
Being rightx_base & leftx_base the starting point for the left and right lines and midpoint is the point in the middle.
xm_per_pix is the conversion x from pixels space to meters

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in my code under the heading `6 Provide an example image`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I know there is much potential in this project. I am aware that radius calculation is not robust but I am very happy with the lane detection.
I found that the radius I calculate in the example image is far from the expected 1km radius. It is still in the acceptable range (400 m). Also, the radius is not stable through the video. To try to fix this I played shrinking a bit the source points in the transformation matrix. I started here because I see that the highest point of the lane is blurry and later when transforming the image, the highest point of the lane is too wide, or it even disappears in the right lane. This ends up increasing the error to calculate the curve. But shrinking the warped image only resulted in an even more unrealistic radius. 
Another option for the improvement of the accuracy of the calculation of the radius could have been taking into account previous frames or adding a low pass filter. 

I feel very happy with my final work after facing this challenge which seemed to me almost impossible at the beginning.
