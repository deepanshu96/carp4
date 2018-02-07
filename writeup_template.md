
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

[image1]: https://github.com/deepanshu96/carp4/blob/master/output_images/i1.png
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I used the distortion coefficients calculated in above steps using the chess board images and applied cv2.undistort() function to the lane images. The undistorted image is shown below:-
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i0.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. First I used the gradient threshold to identify the lane lines. I used 4 types of gradient thresholds namely magnitude threshold, direction threshold and absolute threshold in x and y direction respectively. After that I combined all the gradient thresholds to produce an image identifying the lane lines(note: I did not use magnitude and gradient threshold in the final image used as only absolute thresholds were giving better results for the lane lines in my project). The combined gradient threshold are implemented via 'combinegrad()' function in my project. Gradient thresholds are shown below:- 
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i2.png)
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i3.png)
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i4.png)

The combined gradient threshold:-
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i5.png)

After the gradient threshold I also applied color transforms to the image in order to obtain lane lines more clearly. I used the S space from HSV color image format and R and G from RGB format. R and G combined together helped detect yellow lines more clearly and S helped in detection of other lines. The color transformed image is shown below :-
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i6.png)

I also applied region_of_interest function to the image produced via combination of colour and gradient threshold to remove the unnecessary details from the image. The region of interest is shown below:-
![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i7.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called 'perspect()'. First I applied the distortion correction to the image, then I applied colour and gradient thresholds and finally I masked the region of interest. After that I applied the perspective transform to get the bird's eye view of the lane lines. I chose source and destination points so that the lines remain parallel in the perspective view. I wrote the following code to apply the perspective transform by first calculating the perspective matrix M and then applying the 'warpPerspective' function :- 
```python
def perspect(imag):
    #print(imag.dtype)
    imag = undist(imag)
    #print(imag.dtype)
    image = fincombine(imag)
    #print(image.dtype)
    image = region_of_interest(image)
    
    rows,cols = image.shape

    pt1 = np.float32([[220,720],[570, 470],[722, 470],[1110, 720]]) 
    pt2 = np.float32([[320,720],[320, 1],[920, 1],[920, 720]])
    #pt2 = np.float32([[0,0],[rows,0],[0,cols],[rows,cols]])
    
    Minv = cv2.getPerspectiveTransform(pt2, pt1)
    M = cv2.getPerspectiveTransform(pt1, pt2)
    dst = cv2.warpPerspective(image, M, (cols, rows), flags=cv2.INTER_LINEAR) 
    return dst, Minv
```

The source and destination points are pt1 and pt2 respectively. 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i8.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified the lane lines using the histogram method in which we first find the highest peaks from the bottom most row and taking that as a starting point we further find the active pixels in a rectangular region. If a shift of more than 50 good or active pixels is noticed, we take the median of those pixels in order to fit them in our next rectangular region. Finally after obtaining all the good left and right lanes pixles we fit a polynomial of degree 2 to our lines and plot them on the image. This is done in the 'findlines()' function in the code. 

Histogram image:-

![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i75.png)

Identified Lane lines image:-

![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i9.png)


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature by first changing the pixel dimensions into metre by using the transform scale mentioned and then fit the coordinates into the formula for calculating the radius of a curve. The curve was fitted in the above step for plotting the lane lines on the image. After that the position of vehicle was calculated by measuring the difference between the center point of lane lines on the image and the center point of the whole image in the x coordinate. 
These steps are performed in the 'findlines()' function using the following code
```
    #Curvature
    #ploty , left_fit, right_fit
    y_eval = np.max(ploty)
    '''
    left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
    right_curverad = ((1 + (2*right_fit[0]*y_eval + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])
    print(left_curverad, right_curverad)
    '''
    ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 # meters per pixel in x dimension
    
    left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
    right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
    
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) /                   
    np.absolute(2*right_fit_cr[0])
    

    #offset calculation
    xMax = result.shape[1]*xm_per_pix
    yMax = result.shape[0]*ym_per_pix
    vcent = xMax / 2
    leftline = left_fit_cr[0]*yMax**2 + left_fit_cr[1]*yMax + left_fit_cr[2]
    rightline = right_fit_cr[0]*yMax**2 + right_fit_cr[1]*yMax + right_fit_cr[2]
    midline = leftline + (rightline - leftline)/2
    centeroff = midline - vcent
    
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step using the 'findlines()' function in which I also plotted the final lane area on the image and used this function to obtain the final video pipeline. The plotted resultant image is shown below:-

![alt text](https://github.com/deepanshu96/carp4/blob/master/output_images/i10.png)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
