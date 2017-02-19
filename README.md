## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

Creating a great writeup:
---
A great writeup should include the rubric points as well as your description of how you addressed each point.  You should include a detailed description of the code used in each step (with line-number references and code snippets where necessary), and links to other supporting documents or external references.  You should include images in your writeup to demonstrate how your code works with examples.  

All that said, please be concise!  We're not looking for you to write a book here, just a brief description of how you passed each rubric point, and references to the relevant code :). 

You're not required to use markdown for your writeup.  If you use another method please just submit a pdf of your writeup.

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Perspective Transform:
-------------------------------------
In order to get a better view of how the lane lines are looking ahead, it is useful to get a bird eye view of the image. In order to achieve that, I defined the source points in the original image and define the destination points that I want the source points to located in the new image space. With the help of getPerspectiveTransform from cv2, it is easy to get the transformation and inverse transformation matrix that transform the image to the different image space. In the images below, the source point are defined by the 4 red circles, and the same four circles show up in the bird eye image as well.
<img width="1002" alt="screen shot 2017-02-18 at 2 22 23 pm" src="https://cloud.githubusercontent.com/assets/22971963/23097318/c663a274-f5e5-11e6-845e-0bb2620305f1.png">
The goal in doing this transformation is to get a bird eye view of the lane lines, this is a much better visualization of the lane lines and it is much easier to do further analysis on. In the original image perspective, none of the lane lines appear to be parallel to one another (which is not the actual case). In the bird view though, it is really easy to see the parallel lane lines.

Using color threshold:
-------------------------------------
Once the bird eye view of the image is obtained, the next step is to identify where the lane lines are potentially located. This section focus on using the color threshold to identify that. One characteristic of the lane lines is that they are either yellow or white. Therefore, by applying a color threshold to the image, I can isolate out the white and yellow lanes while ignoring the other distraction. However, this technique is sensitive to the lighting of the condition, as the same color can appear differently in different light condition. The threshold that I ended up took many tuning in hope to cover as many condition as possible. Below are the result to isolate the lane lines based on color threshold.
<img width="999" alt="screen shot 2017-02-18 at 2 52 02 pm" src="https://cloud.githubusercontent.com/assets/22971963/23097485/d8445296-f5e9-11e6-8256-3cc8e26a2b84.png">

Using gradient threshold:
-------------------------------------
In many cases, using only color threshold is not sufficient enough to estimate where the lane lines are located, especially in varying light condition. Another technique that I am using is the gradient threshold. The technique involve the use of sobel filters, in which the filter is taking the derivative of the pixel. The resulting image will show the places in the image where there are rapid change (an edge). The following images show the result of using the sobel filter and applying a threshold to it (Sobel filter in x-dir (sx), Sobel filter in y-dir (sy), mag of sx and sy (mag), dir of sx and sy (dir)).
<img width="996" alt="screen shot 2017-02-18 at 3 29 40 pm" src="https://cloud.githubusercontent.com/assets/22971963/23097691/1c2e529a-f5ef-11e6-9e67-91d4e7c2a8a8.png">
After getting these results, I then combined them into one image and use that as an estimation on where the lane lines are potentially located at. In this case, I ingored the result from sy as I don't believe it will any additional information.
<img width="993" alt="screen shot 2017-02-18 at 3 41 59 pm" src="https://cloud.githubusercontent.com/assets/22971963/23097735/d16c67c2-f5f0-11e6-9235-60261bf6d6d5.png">

Combining gradient and color threshold:
-------------------------------------
Now that there are 2 methods to estimate where the lane lines are located, it is time to combine both threshold and get a more accurate estimate of the lane lines. Even though, there will still be noise in the image that is not the lane lines, but it is a good starting point to perform further analysis.

<img width="988" alt="screen shot 2017-02-18 at 3 50 35 pm" src="https://cloud.githubusercontent.com/assets/22971963/23097771/07f6bc1a-f5f2-11e6-819a-07f57c8c66ca.png">

Lane finding using histogram:
-------------------------------------
Now that there is a image that provide an estimate on where the lane lines are, it is time to find where the locate is actually located at. To do this, I used a histogram method and determined the peak (point with the most pixel) in the x-axis.
<img width="999" alt="screen shot 2017-02-18 at 4 19 30 pm" src="https://cloud.githubusercontent.com/assets/22971963/23097929/134805ac-f5f6-11e6-9412-dafdf0c410cc.png">
In looking at the histogram, there are 2 peaks (one for the left lane and one for the right lane) that are more apparent and this is a good place to start the lanes search in the image.

Once the base points are identified, I can now deploy the window sliding method starting from the base point and identify the lanes in the image. The window sliding method breaks the image down into n sections in the y-axis. Using a window of a specific size, the window will continue to slide up and look for the where the lane lines are located. The image below shows the result of the window sliding method with 9 windows.
<img width="988" alt="screen shot 2017-02-18 at 5 17 10 pm" src="https://cloud.githubusercontent.com/assets/22971963/23098236/1e7f8334-f5fe-11e6-8d40-3cb47734d873.png">

Every pixal within the 9 windows (indicated in red) are considered a good candidate of where the lane lines are located. 

Fitting a second order polynomial:
-------------------------------------
Now that all useful pixels are identified, a second order polynomial function (Ax^2 + Bx + C) can be fitted to describe the lane lines. 

<img width="987" alt="screen shot 2017-02-18 at 5 34 44 pm" src="https://cloud.githubusercontent.com/assets/22971963/23098378/9375177e-f600-11e6-8154-b2b7fef1443a.png">

Overlaying Images:
-------------------------------------


