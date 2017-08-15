# Advanced Lane Detection

---

[//]: # (Image References)

[chessboard1]: ./chessboard1.PNG "Chessboard"
[chessboard2]: ./undistorted1.PNG "Chessboard undistorted"
[image1]: ./original1.PNG "Regular"
[image2]: ./image1.PNG "Undistorted"
[image3]: ./binary1.PNG "Binary"
[image4]: ./warped1.PNG "Warped"
[image5]: ./windows1.PNG "Windows"
[image6]: ./lanes1.PNG "Output"
[video1]: ./project_video.mp4 "Video"

#### This is classwork for the Udacity self driving car nanodegree. Here is the [Rubric](https://review.udacity.com/#!/rubrics/571/view) for this project.

#### In my [code](https://github.com/FreedomChal/advanced_lane_detection/blob/master/P4.ipynb), a much more detailed description can be found of how my code works 

## Camera Calibration

First, to calibrate the camera, I used opencv's cv2.findChessboardCorners and cv2.calibrateCamera to calibrate the camera on chessboard images, then cv2.undistort using the matrix and destination points returned by cv2.calibrateCamera to undistort images.

#### Original:
![alt text][chessboard1]

#### Undistorted:
![alt text][chessboard2]

## Pipeline

### Undistortion

First I undistorted the image using the matrix and destination points from the camera calibration.

#### Original (you can ignore the size difference):
![alt text][image1]

#### Undistorted:
![alt text][image2]

### Thresholding

#### The colorspace I used was HSV

For thresholding, I did thresholding for sobel, a combined thresholding for saturation and hue, and thresholding for value.

#### Binary image:
![alt text][image3]

### Perspective Warping

For the perspective warping, I used cv2.getPerspectiveTransform. The source and destination points I found worked well are as follows:

| Source        | Destination   |
|:-------------:|:-------------:|
| 0, 700        | 0, 720        |
| 1280, 700     | 1280, 720     |
| 557, 450      | 0, 0          |
| 723, 450      | 1280, 0       |

#### Perspective warped:
![alt text][image4]

### Sliding Windows

Next, to find lane line pixels, two sliding windows go across the y values of the warped binary image, to find the left and right lane line pixels, and move to the nearest x value with a large number of non blank pixels.

#### Sliding window locations:
![alt text][image5]

### Line fit and final results

First, I do numpy.polyfit on the sliding window center locations to get second degree polynomials retpresenting the lane lines. Then I convert the polynomial coefficents returned by np.polyfit to x values of the line for each y value in the image. I then get the radius of the lane line polynomial, the distance the veichle is from the center of the lane, add them to the image in text, then do a sanity check to make sure the lane does not change suddenly. I then overlay green on the lane in between the lane lines using code from [here](https://github.com/gardenermike/finding-lane-lines-reprise/blob/master/lane_lines.ipynb). Then I revert the perspective back to normal, add the lane image to the plain undistorted image, then I'm done!

#### Image with lane, radii, and veichle distance from the center of the lane:
![alt text][image6]

---

### Video

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

At first, the sanity check worked somewhat differently, as it checked if the radii were consistent instead of the lane lines from the veichle, which changed. The main problem I have encountered is the sanity check causing the lane lines to not change at all even when the actual lane is somewhere else, then instantly change to a more correct location, causing the video to be jumpy. This could possibly be solved by making it so a new lane location caries more weight if the last predicted lane location was simmilar, even if the lane location that has actually been used for a while is somewhat different.
