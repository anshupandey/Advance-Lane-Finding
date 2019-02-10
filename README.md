# Advance-Lane-Finding


    "# **Advanced Lane Finding Project**"
    "### The goals / steps of this project are the following:\n",
    "\n",
    "* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.\n",
    "* Apply a distortion correction to raw images.\n",
    "* Use color transforms, gradients, etc., to create a thresholded binary image.\n",
    "* Apply a perspective transform to rectify binary image (\"birds-eye view\").\n",
    "* Detect lane pixels and fit to find the lane boundary.\n",
    "* Determine the curvature of the lane and vehicle position with respect to center.\n",
    "* Warp the detected lane boundaries back onto the original image.\n",
    "* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.\n"


## Step 1 - Camera Calibration"
    "In the cell number 3 of my ipynb file this step is implemented\n",
    "For camera calibration, i have used the following steps - \n",
    "1. empty list for storing imagepoints and objectpoints is created\n",
    "2. objectpoint objp is prepared of the size of chessboard which is 9*6\n",
    "3. Using os, I created list of image names, then in a for loop below steps are implemented - \n",
    "    - reading the image and converting it into grayscale format\n",
    "    - Using function cv2.findChessboardCorners() the corners are detected\n",
    "    - the corners are the appended into the imagepoints list and for the same imag objp is appended into objectpoints list\n",
    "    - finally we got the objectpoints and imagepoints which can be later used for undistorting images\n",
    "    - at the end in order to be sure that the corners detected are correct, corners on the chessboard are drawn\n",
    "\n",
    "</br>\n",
    "Some of the examples of detected corners are given below:    "


    "<img src='output_images/corners_drawn1.png'>\n",
    "<img src='output_images/corners_drawn2.png'>\n",
    "<img src='output_images/corners_drawn3.png'>"
   
   
    "In cell number 4 by using the function cv2.calibrateCamera() using the above collected imagepoints, objectpoints and the size of the image the camera matrix mts and the distortion coefficient dist has been calculated"
   
   
    "## Step 2 - Distortion Correction"
   
   
    "The function with name cal_undistort is created whcih accepts the image, and some collected values from above steps like objectpoint and imagepoints and returns the undistorted image is implemented in cell 4 of the code.\n",
    "\n",
    "\n",
    "The camera matrix mts and the distortion coefficient from above step is used here for removing the distortion in the image.\n",
    "\n",
    "For this the function cv2.undistort from openCV is used.\n",
    "One of the example of removing distortion on the chessboard image is\n",
    "attached below:"
   
   
    "<img src='output_images/undistort_output.jpg'>"
   
   
    "Here the function is used and applied on the lane image and the undistorted and orignial image is attached."
   
   
    "<img src='output_images/undistort_output_test_image.jpg'>"
   
   
   
    "## step 3 - Creating thresholded binary Image"
   
   
    "This step is implemented in cell 6 of the notebook.\n",
    "there are many techniques of getting the binary threshold image which contains the lane pixels, Based on my experience of learning from the course I have used following technique\n",
    "\n",
    "1. First the image is converted from RGB space to HLS space, then S component of the image is taken spearately\n",
    "2. Usign sobel technique, the derivative of the image is obtain on x axis and then mapped to values between 0 to 255 as the image of type 8 bits.\n",
    "3. Gradient Thresholding - thresholding on the derivative image (x gradient) is applied with min value =20 and mx value = 100 that means, pixel values greater than 20 and less than 100 are taken and are represented as white pixels \n",
    "4. Color Thresholding - thresholding on the s channel image is applied with range between 20 to 100\n",
    "5. the above two outputs from Gradient thresholding and Color thresholding is combined using OR logic\n",
    "\n",
    "\n",
    "One of example of a lane image thresholded is attached below - m"
   
   
   
    "<img src='output_images/thresh-x-example.png'>"
   
   
   
    "## step -4 Prespective Transform"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "In the cell number 9 of the code the prepesctive transform of the image is implememted\n",
    "\n",
    "A function with the name warper is created, the function does following steps\n",
    "\n",
    "1. undistoring the image using the cal_undistort() function created in step 2 which accepts the image, previously created objectpoints and imagepoints from step 1\n",
    "2.  follwoing source and destination points are used to apply prespective transform\n",
    "<br>\n",
    "<table>\n",
    "<b><td><b>source point</td><td><b>Destination points</td>\n",
    "<tr><td>[180,710]</td><td>[280, 720]</td></tr>\n",
    "<tr><td>[1150,710]</td><td>[1000, 720]</td></tr>\n",
    "<tr><td>[680,450]</td><td>[1000, 0]</td></tr>\n",
    "<tr><td>[600,450]</td><td>[280, 0]</td></tr>\n",
    "</table>\n",
    "    \n",
    "3. Using the function cv2.getPerspectiveTransform() and cv2.warpPerspective the warped image is obtained.\n",
    "4. using the dst and src, the inverse matrix is also obtained which will be used to map back to original image once lane pixels are properly detected.\n",
    "\n",
    "An example of the prespective transformed image is attached below"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<img src='output_images/wrapped_straight_lines.png'>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Example of prespective transform on binary thresholded image is attached here:"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<img src='output_images/warped_example-output.jpg'>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## step 5 - Finding lane line pixels and polynomial fit"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "In the cell 11 of the notebook this step is implemented.\n",
    "below steps are taken to identify laneline pixels and then performing a polynomial fit\n",
    "\n",
    "1. the first step is taking the histogram of binary halg image because in the binary half the cordinates where the actual lane line pixels lie we should get high peak in the histogram, taking bottom half will remove the noise from the top half\n",
    "2. Now the peaks of left and right half is calculated using numpy.argmax function\n",
    "3. the image is divided into 9 windows with a widow margin of 100, the threshold for minimum number of pixels to be observed in a window is set to 50\n",
    "4. All the nonzero pixel positions are identified and stored in nonsserox, nonzeroy\n",
    "4. using a for loop running through all the windows i.e. 9 windows, the boundaries of the window are extracted using margin and thene nonzeros pixels are identified in respective window and finally the left and right lane indeices corresponding to the nonzero pixels in respective windows are appended into left_lane_inds and right_lane_inds\n",
    "5. finally the ledt and right line pixel positions are identified.\n",
    "<br><br>\n",
    "    Now as we have final left are right lane line pixel positions its time to fit the polynomial line on it. for this step a function with the name fit_polynomial is created.\n",
    "    \n",
    "In the fit_polynomial function, a quadratic equation which is \n",
    " \n",
    " A*y^2 + B*y + c \n",
    " \n",
    " is fit\n",
    " where \n",
    " first left_fit and right_fit is calculated using numpy.polyfit on output of below step\n",
    " and  A = left_fit[0], B = left_fit[1] and C = left_fit[2] is used.\n",
    " \n",
    " left lane pixels and right lane pixels are coloured red and blue respectively.\n",
    " the polynominal is plotted with yellow color.\n",
    " \n",
    " The output of above step is attached here - "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<img src='output_images/lane_pixel_detected.jpg'>\n",
    "<img src='output_images/lane_pixel_detection_wraped_image.png'>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## step 6 - Calculating the radius of curvature and distance from lane"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "to convert the radius of corvature into meters from pixel following factor is used - \n",
    "\n",
    "ym_per_pix = 30/720 # meters per pixel in y dimension\n",
    "xm_per_pix = 3.7/700 # meters per pixel in x dimension\n",
    "\n",
    "following formula is used to calculate the radius of curvature on left and right lane \n",
    "\n",
    "<h4>R = ((1 + (2*A*y*ym + B)**2)**1.5) / |2*A|</h4>\n",
    "\n",
    "further the value of left curvature and right curvature is averaged.\n",
    "\n",
    "TO calcualte the position of the car firs the center of the lane is obtained and then the center of the image is calculated and the distance between the center of the image and center of lane is considered as the distance of the vehicle from the center.\n",
    "\n",
    "<b>Distance of Vehicle from center = Center of Image - Center of the lane\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "In cell number 13\n",
    "\n",
    "To get the lane boundaries on the original image, using cv2.fillpoly() function the lanes are drawn on the watped blank image\n",
    "then the blank image is warped back to the orignial image using inverse prespective matrix\n",
    "the result is then combined with the orignial image\n",
    "\n",
    "<img src='output_images/example_output-lane-area.png'>\n",
    "<img src='output_images/example_output-lane-area2.png'>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Possible Improvements - "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "1. Fiting the polynomial function is every frame can be avoided by estimating it for upcoming frames\n",
    "2. the factor for pixel to meter conversion can be more precisely calculated.\n",
    "3. THe write can include more theoretical explanation then program explanation."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Faced Challenges - "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "1. While understanding how to take destination points while performing prespective transform, later on the questions asked by other learners in past helped.\n",
    "2. How to take the 30m range for source point was a BIG deal and still I have doubts on it\n",
    "3. Calcultion of final radius of curvature from left and right is what I thought of taking average based on intution but got it confirmed by reading some questions.\n"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.7"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
