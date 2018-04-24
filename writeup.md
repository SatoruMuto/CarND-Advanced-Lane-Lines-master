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


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./CarND-advanced_lane_line.ipynb"  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![](./image_file/camera_cal_result.PNG?raw=true)
  

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Once I get camera calibration coefficient from above step, I can simply apply  `cv2.undistort()`  to vehicle on-board camera image.
here is one of the test images like this one:
![](./image_file/onborad_camera_cal_result.PNG?raw=true)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of HLS color transform and x-gradient thresholds to generate a binary image (in the 23 code cell of the Jupyter notebook).  Here's an example of my output for this step. 
I have tuned thresholds as blow in order to extract lines only as possible as I can.

`s_thresh=(150, 255) # HLS s channel
sx_thresh=(50, 150) # HLS l channel (without threshold) with x-gradient `

![](./image_file/combined_binary_image.PNG?raw=true)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:  


`python
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
`  

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![](./image_file/source_points_plot.PNG?raw=true)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In order to find lane line pixcels, I have used slinding window serch method. Basically I split y axis to 9 reasion, and find 1st and 2nd peak of each reasion by histgram, those 2 peaks should be good indicater of x-position of 2 lane lines. 

in below picture, there are red line and blue line. Those are 2 peaked area that were detected as most possible position of 2 lane lines. Serching window were also shown as green box.

Then I used numpy polyfit function to get 2nd order polynominal.
The found poly line was plotted as orange line. 

![](./image_file/sliding_window.PNG?raw=true)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature is calculated for left lane and right labe separately. The code is in cell No 29, `map_lane()` function in Jupyter notebook.

1st I need to define real metric and pixcel relation ship. I took metric number from US standard of lane line road marking. 
`ym_per_pix = 3.05/110 # US standard lane line length is 10 ft = 3.05 meters`
`xm_per_pix = 3.7/640 # International highway standard lane width is 3.7 meters`

then 
`left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])`   
`right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])`   
   


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell No 29 in my Jupyter notebook `carnd-term1_project4.ipynb` in the function `map_lane()`.  Here is an example of my result on a test image:

![](./image_file/detected_area_plot.PNG?raw=true)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I tuned parameters on combined binary image, warper, etc in order to get good result. especially to clean-up image to extract clear lane line by combined binary image method was key. However it seems that it is not completely robust on shadow, and road surface change point (such as concreat to Asphalt, at bridge to normal road boundary).  

It does not continue long time, so if I apply filtering method to outliner, this may be improved. If I store 5 past lane line and rolling mean those value to calculate lane line, this should be improved. 
