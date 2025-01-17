## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, the goal is to create robust pipeline that can identify the the lane markings in the video.

The pipeline that I ended up with broken down into the following sections:

* Camera Calibration
* Perspective Transorm
* Using color threshold
* Using gradient threshold
* Combining color and gradient threshold
* Lane finding using histogram
* Fitting a second order polynomial
* Overlaying Images
* Calculating curvature

Camera Calibration:
-------------------------------------
Each camera lens have its own unique curvature and therefore will distort the image differently. When working with images, it is important to undistort the image so we can accurate get the respresentation and the correct information of the scene. With the help of openCV, this process is easy. The concept behind this is to find a known shape so we will be able to detect and correct the distortion. One of the most common object to use is a chessboard because the pattern makes it easy to detect. To get an accurate transform, we often want at least 20 different images of chessboard, taken from different angle and distance. Below show an image where openCV found the corner of the chessboard. The same transform is then used in the image on the road.

<img width="1267" alt="screen shot 2017-02-27 at 8 20 06 pm" src="https://cloud.githubusercontent.com/assets/22971963/23391493/3ba03a16-fd2a-11e6-83d7-5c1cb388d617.png">

<img width="996" alt="screen shot 2017-02-27 at 8 34 46 pm" src="https://cloud.githubusercontent.com/assets/22971963/23391793/3a56a3a0-fd2c-11e6-95a5-2916cb0d04ba.png">

Perspective Transform:
-------------------------------------
In order to get a better view of how the lane lines are looking ahead, it is useful to get a bird eye view of the image. In order to achieve that, I defined the source points in the original image and define the destination points that I want the source points to located in the new image space. With the help of getPerspectiveTransform from cv2, it is easy to get the transformation and inverse transformation matrix that transform the image to the different image space. The image below shows the undistorted image and the corresponding bird eye transformation.

<img width="1001" alt="screen shot 2017-02-27 at 8 48 43 pm" src="https://cloud.githubusercontent.com/assets/22971963/23392084/29c4ceb6-fd2e-11e6-9d3c-3c4195a67dcc.png">

The goal in doing this transformation is to get a bird eye view of the lane lines, this is a much better visualization of the lane lines and it is much easier to do further analysis on. In the original image perspective, none of the lane lines appear to be parallel to one another (which they should be). In the bird view though, it is easy to see that the lanes appear to be somewhat parallel.

Using color threshold:
-------------------------------------
Once the bird eye view of the image is obtained, the next step is to identify where the lane lines are potentially located. This section focus on using the color threshold to identify that. One characteristic of the lane lines is that they are either yellow or white. Therefore, by applying a color threshold to the image, I can isolate out the white and yellow lanes while ignoring the other distraction. However, this technique is sensitive to the lighting of the condition, as the same color can appear differently in different light condition. The threshold that I ended up took many tuning in hope to cover as many condition as possible. Below are the result to isolate the lane lines based on color threshold.

<img width="1010" alt="screen shot 2017-02-27 at 9 00 15 pm" src="https://cloud.githubusercontent.com/assets/22971963/23392341/cf237da2-fd2f-11e6-834a-a24730b59e62.png">

Using gradient threshold:
-------------------------------------
In many cases, using only color threshold is not sufficient enough to estimate where the lane lines are located, especially in varying light condition. Another technique that I am using is the gradient threshold. The technique involve the use of sobel filters, in which the filter is taking the derivative of the pixel. The resulting image will show the places in the image where there are rapid change (an edge). 4 uses of sobel filter are used and a threshold to each (Sobel filter in x-dir (sx), Sobel filter in y-dir (sy), mag of sx and sy (mag), dir of sx and sy (dir)). These results are then combined into one image and used as an estimation on where the lane lines are potentially located at. In this case, I ingored the result from sy as I don't believe it will any additional information.

<img width="992" alt="screen shot 2017-02-27 at 9 07 47 pm" src="https://cloud.githubusercontent.com/assets/22971963/23392553/4c1e51c8-fd31-11e6-90d2-c47b4220a340.png">

Combining gradient and color threshold:
-------------------------------------
Now that there are 2 methods to estimate where the lane lines are located, it is time to combine both threshold and get a more accurate estimate of the lane lines. Even though, there will still be noise in the image that is not the lane lines, but it is a good starting point to perform further analysis.

<img width="998" alt="screen shot 2017-02-27 at 9 16 57 pm" src="https://cloud.githubusercontent.com/assets/22971963/23392645/19e2521c-fd32-11e6-92a2-f8c0b3028e80.png">

Lane finding using histogram:
-------------------------------------
Now that there is a image that provide an estimate on where the lane lines are, it is time to find where the locate is actually located at. To do this, I used a histogram method and determined the peak (point with the most pixel) in the x-axis.

<img width="991" alt="screen shot 2017-02-27 at 9 23 08 pm" src="https://cloud.githubusercontent.com/assets/22971963/23392791/f81af8cc-fd32-11e6-924c-edbafcef9dbc.png">

This is also the place where the first sansity check is applied. One thing that is assumed here is that the camera is mounted in middle of the car. This mean that there is an expected range and location in where the lane will start. In this case I decided to have an expected start point to be between midpoint +/- 100 and midpoint +/- 200 where + is for the right lane and - is for the left lane. In looking at the histogram, there are 2 peaks (one for the left lane and one for the right lane) that are in the expected range and this is a good place to start the lanes search in the image.

Once the base points are identified, I can now deploy the window sliding method starting from the base point and identify the lanes in the image. The window sliding method breaks the image down into n sections in the y-axis. Using a window of a specific size, the window will continue to slide up and look for the where the lane lines are located. The image below shows the result of the window sliding method with 9 windows.

<img width="990" alt="screen shot 2017-02-27 at 9 36 33 pm" src="https://cloud.githubusercontent.com/assets/22971963/23393048/d6d0a44e-fd34-11e6-85da-e508256decf1.png">

Every pixal within the 9 windows are considered a good candidate of where the lane lines are located. 

Fitting a second order polynomial:
-------------------------------------
Now that all useful pixels are identified, a second order polynomial function (Ax^2 + Bx + C) can be fitted to describe the lane lines. 

<img width="992" alt="screen shot 2017-02-27 at 9 47 51 pm" src="https://cloud.githubusercontent.com/assets/22971963/23393244/6b93d97e-fd36-11e6-97ea-a85fc5183e33.png">

Overlaying Images:
-------------------------------------
Now that there is an estimate of the potential location of the lines, the next step is to map that back to the original image. First, an image of that shows where the line is is created. To create that image, all the pixal between the 2 predicted lane are filled in.

<img width="995" alt="screen shot 2017-02-27 at 9 57 52 pm" src="https://cloud.githubusercontent.com/assets/22971963/23393455/d1f55e80-fd37-11e6-8a9a-1496ec31261c.png">

Calculating curvature:
-------------------------------------
Another way to check the accuracy of the estimation is to look at the curvature of the lanes to see if it make sense. The equation of radius of curvature is given by:

<img width="186" alt="screen shot 2017-02-18 at 8 44 19 pm" src="https://cloud.githubusercontent.com/assets/22971963/23099418/16c66136-f61b-11e6-93fc-865b2c0dda6c.png">

where A and B are the coefficient calculated above, and y is the value of the bottom of the image (height of image).
After calculating the radius, the value need to be convert into meter because so far the calulation is taking pixal as an input. (x: 3.7 meter per 700 pixal, y: 30 meter per 720 pixal)

<img width="991" alt="screen shot 2017-02-27 at 10 02 34 pm" src="https://cloud.githubusercontent.com/assets/22971963/23393555/7b0dd060-fd38-11e6-8628-606a2d4b7a52.png">

Video Result:
-------------------------------------
https://youtu.be/YwPZ2TFLY4U

Reflection:
-------------------------------------
Currently, this pipeline is still not robust to lighting change and it will require further tuning of the parameter. A few thing that I am planning to work on is to apply the following sanity checks to make the pipeline more robust.
* weight the contribution for color threshold image and the gradient image. This will allow the pipeline to get a more accurate image to process on.
* Check the slope of left and right lanes. Since the lanes should be roughly parallel to each other.
* Check the curvature of each lane, reject curvature with unrealistic number
* Check the basepoint of the lane, from frame to frame there shouldn't a significant change
* The number of nonzero pixal in the image, if there is a significant increase in pixal, there it is safe to assume that there is a lighting change and the image should not be trusted


