
# **Advanced Lane Finding Project**

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
[image1]: ./images/camera_calib.png "Undistorted"
[image2]: ./images/straight_lines1.jpg "Test Image"
[image21]: ./images/1_Undistorted_Image.png "Undistorted Example"
[image3]: ./images/1_Threshold_Image.png "Binary Example"
[image4]: ./images/1_ROI_Warped.png "Warp Example"
[image5]: ./images/1_Draw_Lines.png "Fit Visual"
[image6]: ./images/3_example_output.png "Output"
[video1]: ./images//project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

---

### Camera Calibration

#### 1. Camera calibration using the OpenCV functions and the calibration images 
Tha Calibration images are provided in the (./camera/cal) and are of chessboard (with 10x7 squares) taken at different angles and distance.  I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here the assumption is that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. So the obecjpoints `objp` is just a replicated array of coordinates of grid(9x6), and `objpoints` will be appended with a copy of it every time successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection (the code for Camera calibration is at line 52 in P4.ipynb).
And then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. 
and applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Image Pipeline (single images)
To find lines in a given image and then to draw lines, each image has been processed as follows:
1. Undistort the Image using Camera Calibration matrix and distortion coefficients
2. Find line x,y co-ordinates using Thresholding (Create binary image)
3. Apply Perspective Transform to unwarp the image
4. Find lines in the warped+thresholded binary image using 
   blind search/previous frame findings
5. Unwarp the image and Draw the lines (calc roc/offset)
   

#### 1. Undistort the image using Camera Calibration matrix and Distortion coefficients

To undistort the image have used the Camera calibration matrix and distortion coefficents calculated using camera calibration as decribed above.
The distortion coefficients correct for the tangential and radial distortion effects that are caused by the real cameras.
Real cameras use curved lenses to form an image, and light rays often bend a little too much or too little at the edges of these lenses. This creates an effect that distorts the edges of images, so that lines or objects appear more or less curved than they actually are. This is called tangential distortion. Another type of distortion, is radial distortion and this occurs when a camera’s lens is not aligned perfectly parallel to the imaging plane, where the camera film or sensor is. This makes an image look tilted so that some objects appear farther away or closer than they actually are. 
Using the OpenCV function `cv2.undistort`, images have been undistorted and following example shows an undistorted image


![alt text][image21]

### 2. Creating a Threshold Image
For generating the binary thereshold image experiemtned with various combination of color space and gradient thresholds
as detecting both white and yellow lines under different lighting conditions was a challenge. First converted the image to different color spaces from RGB to HLS, Lab, HSV, etc. to get different attributes of the image color space and then experimented with them.
Using x gradient helped detect the lines oriented in vertical direction but under low-light conditions (shadows, etc.)
posed difficulty. So used saturation compoment (from HLS) to detec the lines under different light conditions but
detection of yellow line posed a problem under shadows. So, finaly used a combination of "b" and "s" components from 
Lab and Luv color spaces along with x gradient successfully detected the White/Yellowlanes lines under different lighting conditions (thresholding steps at lines 330 through 360 in `P4.ipynb`).  Here's an example of my output for this step.  (note: this is one of the test images)

![alt text][image3]

### 3. Apply Perspective Transform and unwarp the image

The code for my perspective transform includes a function called `corners_unwarp()`, which appears in lines 411 through 422 in the file `P4.ipynb` (1st cell of the IPython notebook `P4.ipynb`).  The `corners_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
# Select the region of interest for Perspective Transform
With the tuning parameters as:
roi_ll_wid = 0.15; roi_lr_wid = 0.89
roi_ul_wid = 0.45; roi_ur_wid = 0.55
roi_ur_hgt = 0.36   # % height of ROI from bottom of Image
roi_dst_voffset = 0 # pixels
roi_dst_hoffset = 320 # pixels

img_height = img_size[0]
img_width  = img_size[1]
#print("width", img_width, "height", img_height)
roi_ll_x = int(roi_ll_wid * img_width)
roi_ll_y = int(img_height)
roi_lr_x = int(roi_lr_wid * img_width)
roi_lr_y = roi_ll_y
roi_ul_x = int(roi_ul_wid * img_width)
roi_ul_y = int(roi_ur_hgt * img_height)
roi_ur_x = int(roi_ur_wid * img_width)
roi_ur_y = roi_ul_y
ver_ll = (roi_ll_x, roi_ll_y)
ver_ul = (roi_ul_x, roi_ul_y)
ver_ur = (roi_ur_x, roi_ur_y)
ver_lr = (roi_lr_x, roi_lr_y)
    
#Source and Destination points
src = np.float32([ver_ul, ver_ur, ver_lr, ver_ll])
dst = np.float32([[hoffset, voffset], [img_width-hoffset, voffset], 
                 [img_width-hoffset, img_height-voffset], [hoffset, img_height-voffset]])
```

This resulted in the following source and destination points:

| Source (x,y)   | Destination (x,y)  | 
|:--------------:|:------------------:| 
| 576, 460       | 320, 0             | 
| 704, 720       | 960, 0             |
| 1139, 720      | 960, 720           |
| 192, 460       | 320, 720           |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

### 4. Detect and Draw lines on the image
Using the binary threshold unwarped image, the lines are detected using a histogram approach. The code for detecting 
lines is described in the function `find_lines_frame()`. The function takes binary warped threshold image along with 
window margin width (window to search around histogram detection) and minpix (minimum pixels to consider detection 
of line pixels within a window). First take a histogram of the bottom half of the binary image and then peaks are 
detected  on either side of the mid-point(image_width/2) in the image. Then the image is divided into n (=9) windows
of equal height and then pixels are searched for within the maring window of window_margin_width(=100) on either side 
of the histogram peak. Once you find # of pixels are greater then minpix(=50), then these pixels are considered as part 
of the line. Then this search process is moved to next higher window in the image and continued for all the windows
in the image. ALl these pixel indices(x,y) are appended and the center around which search is performed is moved as 
we move from bottom to top of the image. 

After detecting the pixel indices for left and right lanes, a line is fitted using the numpy function `np.polyfit()` 
that takes x & y indices and polynomial degree(=2). Then we generate x and y values of the line to plot vertically 
(y=0, 719) using the polynomial. Then the line is drawn on the binary image showing the pixels used for detection 
and the lines (fitted using the generated polynomial). The code for plotting the lines is given in the function `plot_finding_lines()`  takes x,y indices, the x,y values for drawing lines.
An example of the image with lines plotted is shown below:

![alt text][image5]

### 5. Calculation of Radius of Curvature(roc) and line base position (offset from center):
The radius of curvature and offset is implemented in the lines 625 in function `calc_roc_offset()`. The function
takes the x and y indices of detected pixels and then fits a polynomial after converting the x and y dimenstions
to real world space. I assumed the scaling of the image in the x and y dimensions comapred to real world space as
follows. Then caluclated the radius of curvature using the polynomial coeeficients as given below. The reference
for this is taken from 
[Radiusof Curvature reference](https://www.intmath.com/applications-differentiation/8-radius-curvature.php)

```python

ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension

# Fit new polynomials to x,y in world space
# Fit a second order polynomial in real world space
# Fit new polynomials to x,y in world space
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
# Calculate the new radii of curvature
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
# Radius of curvature is in meters
print('roc in meters:',left_curverad, 'm',right_curverad, 'm')

```

The offset is calulated using the x-intercept of the detected lines (left and right) and then calculating the 
horizontal distance of these points from the assumed mid-point(center of the image image_width/2). And then
these distances are averaged to get the offset from center and the left or right positions to center in the
same function (`calc_roc_offset()`) in the pixel dimensions. Then the value is scaled to real world space using
xm_per_pix scaling factor.

### 6. Plotting the detection back to original image.

This is implemented in the `draw_lines()` function that takes undistorted image, binary warped image,  inverse Perspective transform matrix and the fitted lines to draw and fill the area within the lines with color(=GREEN). The function 
first creates a color warped image and then fills the pixels between the fitted lines with color. Then the undirtoed image is warped back to original image using the inverse Persepctive transform matrix and merged with the color warped image
using the OpenCV function (`cv2,addWeighted()) with wieghting factors to merge and display both the images. An example
of the output shown below with Radious of Curvature and Offset displayed on the image using (`cv2.putText()` function)

![alt text][image6]

---

## Pipeline (video)

### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [project_video](./images/out_project_video.mp4) and

Here's a [challenge_video](./images/out_challenge_video.mp4)

Pipeline function is implemented in the `Process_Video()` function and all the steps described above are performed for each image. For optimization purpose, used the previous frame search results to help detection in the current frame. And to 
do that stats for each line are maintained in `Line()` class for each successful detection. The average of the last few
polynomial fits is used to account for jumping of the line detection values. Also a sanity check is performed whenenver
lines are detected to confirm that the lines are valid. This is done by calculating the horizontal distance between the
lines at three different points (their x-values at 0, 720/2 and 719) with some tolerance. Finally the lines are drawn on 
the unwarped images and saved to video file.

---

## Discussion

### 1. Summary of challenges and potential improvements:
Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

While doing this project, I spent quite a bit of time to find the optimum tuning parameters for various functions like
setting Source/Destination points for Perspective transform, thresholds for detecting the lines under various conditions
etc. One of the fundamental issues has been trying to detect the yellow and white lanes under varying lighting conditions. And on research found using Lab space could help with yellow lines and Luma to help with white lines detection. After that made sure both the lines are detected using the test images from challenge video also. Along with that to optimize the search
using the previous frame and to check if lines are valid had to implement sanity check function (that was difficult
using roc and slope) using horizontal distance approach. 
The pipeline worked well with the input video `project_video.mp4` to detect the lane lines and calculating the ROC and offset.
But had difficulty with challenge video for some  images where there were strong shadows and false lines(cracks/markings on the road). I would like to improve the search of lines in a video frame using the previous frame line detection by
restricting the search area using the car speed (to calculate the expected maximum lane change roc and direction) and
improve the search. And also to effectively detect the lanes by making a prediction of the lanes in next image using
the fitted lines in previous successful images.
It was good exposure to various techniques used in this prject to detect lanes in the image and also to realize the importance of camera calibration for successful detection.

