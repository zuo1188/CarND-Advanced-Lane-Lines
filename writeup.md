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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/undistort.png "Road Transformed"
[image3]: ./examples/ortho.png "Binary Example"
[image4]: ./examples/binary.png "Warp Example"
[image5]: ./examples/polyfit.png "Fit Visual"
[image6]: ./examples/fine.png "Output"
[image7]: ./examples/rectange.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

calculate distortion parameters of the camera using chess board images



![alt text][image2]

#### 2. Generate ortho view image

`src` and `dst` are source and destination points,using `cv2.getPerspectiveTransform` to get and translation matrix then 
implement it to the input image
```
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
                  
M = cv2.getPerspectiveTransform(src, dst)

warped = cv2.warpPerspective(img, M, (w,h), flags=cv2.INTER_LINEAR)
```

The image below demonstrates the results of the perspective transform:
![alt text][image3]
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
#### 3. ortho image to binary

I tried several types of edge detection algorithms to enhance lane pixels of the image
![alt text][image4]

finally used a combination of HSL-L channel and LAB-B channel to polyfit
#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

firstly, search two lines pixel base root of the image by sliding a rectangle step by step . collecting non-zero pixels of the region for fit.
secondly, useing numpy.polyfit to fit a Second order polynomials
![alt text][image7]


![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in block IN:[116]  in my code `project.ipynb`

`curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])`


y_meters_per_pixel is the factor used for converting from pixels to meters.
fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based

`lane_center_position = (r_fit_x_int + l_fit_x_int) /2`
center_dist = (car_position - lane_center_position) * x_meters_per_pix
`r_fit_x_int` and `l_fit_x_int` are the root point the right and left fits where y==719, `car_position` is root center of the image: `image_width/2`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in block `In [123]` in my code in `project.ipynb` in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Because the I used hsl-l channel for edge detection ,so the algorithm do not work well when lighting or shadow changes.
The classical edge detection method is hard to solve the problem,I wish deep leaning can solve it.But it need many lane mark labling data.

Here's the result:

[normal video](./project_video_output.mp4)

[challenge_video](./challenge_video_output.mp4)

[harder_challenge_video](./harder_challenge_video_output.mp4)


---
