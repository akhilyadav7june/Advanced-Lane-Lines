## Advanced Lane Finding

The goals / steps of this project are the following:

1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Apply a perspective transform to image ("birds-eye view").
4. Use color transforms, gradients, etc., to create a thresholded binary image.
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
9. Pipeline section
10. Video Output

### 1. compute the camera calibration using chessboard images

Here i used all the 9X6 chessboard images to obtain objects and image points.And i used `cv2.calibrateCamera` function to find the camera matrix and distortion coefficients
   
### 2. Applying a distortion correction to all given test images

Here i used `cv2.undistort` to undistort the distored image


### 3. Apply a perspective transform to image ("birds-eye view").

I changed the strategy from this point. Previously i was first converting the raw image into a binary threshold image then taking the perspective transform. But after lot of experiment in color spaces and gradient method. I didn't get satifactory result. The main problem with farther lane pixels in the image which are not identified when there is dark in image or too far.
So whenever we find binary threshold image result was not good. So i decided to first transform image in perspective view. Which gives all the lane pixel clearly in vertically distributed on the image. And after finding binary image result is also good.

Here i used below four source and destination points.
This i found from straight line image (straight_lines1.jpg) given in test_images folder

    imshape[0] = height of the image
    imshape[1] = width of the image

Source points:

    bottom_left = [0+130,imshape[0]]
    bottom_right = [imshape[1]-80,imshape[0]]
    top_left = [imshape[1]//2-85,imshape[0]//2+110]
    top_right = [imshape[1]//2+95,imshape[0]//2+110]

Destination points:

    bottom_left = [0+250,imshape[0]]
    bottom_right = [imshape[1]-300,imshape[0]]
    top_left = [0+250,0]
    top_right = [imshape[1]-300,0]]
    
### 4. Use color transforms, gradients, etc., to create a thresholded binary image.

In this step i done a lot of experiment with different color spaces like RGB,HLS and gradients.
After lot of experiment i decided to use S channel (threshold = (170,255)) from HLS color space. Which gives a good result in almost all the images together with sobelx gradient (threshold = (50,100)). It worked good until i got the issue in project_video. In one of the location where tree shadow appears this method failed. 
Then i again experimented with LUV,LAB color space.

    I choose B channel from LAB color space with threshold (0,110). I found that this gives good result in all images even on dark one. But it works well on yellow lane only.
    
    I also choose L channel from LUV color space with threshold (200,255). This worked for white lane.
    
So these two channel is sufficient to find the lane lines from all the images. Then i removed the S channel and sobelx gradient from my code.

### 5. Detect lane pixels and fit to find the lane boundary.

This function is used to find the left and right lane pixels on warped binary image
Here i am identifying peaks in a histogram of the image to find the location of left and right lane.
then identifying all non zero pixels around histogram peaks using the numpy function `numpy.nonzero()`. And using `numpy.polyfit()` function fitting a polynomial to each lane.

### 6. Determine the curvature of the lane and vehicle position with respect to center.
      
Here i am calculating radious of curvature in metrs for left and right lane using the following code
first i converted pixels into meters then using below code i calculated left and right curvature
  
    ym_per_pix = 30/imshape[0]
    xm_per_pix = 3.7/imshape[1]

    ploty = np.linspace(0,imshape[0]-1,imshape[0])

    left_fit = np.polyfit(lefty*ym_per_pix,leftx*xm_per_pix,2)
    right_fit = np.polyfit(righty*ym_per_pix,rightx*xm_per_pix,2)
    
    y_eval = np.max(ploty)
    
    left_curvered = ((1 + (2*left_fit[0]*y_eval*ym_per_pix + left_fit[1])**2)**1.5)/ np.absolute(2*left_fit[0])
    right_curvered = ((1 + (2*right_fit[0]*y_eval*ym_per_pix + right_fit[1])**2)**1.5)/np.absolute(2*right_fit[0])

### 7. Warp the detected lane boundaries back onto the original image.

Here i am converting the warped image back to the original image.

### 8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Here i am plotting the polynomials on to the warped image, fill the space between the polynomials to highlight the lane and then applying inverse perspective trasformation to unwarp the image from birds eye back to its original perspective. And printing the distance from center and radius of curvature on to the final image.

### 9. Pipeline section

Implemented full pipeline process in this section. The function takes raw image and return the image with lane detected on it with radious of curvature and center offset on it.

### 10. Video Output

In this section i am applying all the pipeline on project_video.mp4

### Discussion
#### For Challenge project issues and improvements
    
1. Previously i started s channel together with sobelx gradient but that not worked in project video where dark shadow appeared. Then i used B channel from LAB color space and L channel from LUV color space. B channel identified yellow lane and L channel identified white lane. These two channel is sufficient to identified lane on the road.
2. Normal project pipeline is not working for challange and harder_challenge video. 
3. The lanes lines in the challenge videos were extremely difficult to detect. They were either too bright or too dull.
4. For the harder challenge i will try to take perspective transform for smaller section. I tried this but then it failed for normal project video. There is one thing which i never tried yet. When no lane detected consider the previous frame. That may be solve my issue for both video.
5. The challenge video has a section where the car goes underneath a tunnel. Whole image is in black.