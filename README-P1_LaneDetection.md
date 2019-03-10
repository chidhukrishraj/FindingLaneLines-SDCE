# **Finding Lane Lines on the Road** 

## Simple lane lines detection using computer vision techniques

### This document provides the brief description on how the lane lines are detected using computer vision techniques

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on the work in a written report

### Abstract
In the growing domain of the autonomous driving, it is essential to know the lane in which the vehicle is driving and which path is to follow on the roads. This lane line finding techniques is a simple way of approach to detect lines using the image processing technique on videos. The algorithm uses different ways of processing the image at first and using the same as reference for the continuous image processing on video. 


### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My work consists of following steps to detect the lane lines.

**Step 1:** To start with the image process, the very first step is to read the image which is provided by the matplotlib library. the same library is used for plotting the image. An image in the digital world is a combination of pixels with range 0 to 255, 0 being the dark(black) color and 255 being the brigh(white) color. But for the image processing it is recommended to minimize the pixel values by converting the image to gray scale image using the function grayscale(img). In OpenCV2, the function cvtColor() is used to convert the image to gray scale. This way we can reduce the image pixel to 0 to 10(approximately). This minimize our effort of calculation.

**Step 2:** The converted image is then used for the edge detection and for this we use the Canny algorithm. The Canny function requires an lower threshold and higher threshold which used used to detect the edges. The Canny algorithm only provides the information between the lower and higher threshold and the rest is not included. In my project I provided the value 100 for lower threshold and 200 for the higher threshold. The function returns the image with the edge detected. 

**Step 3:** But our interest is not in the edges in the whole image. We need to find out the lanes and for this we provide only a particular region on which the image returns the edges detected. For this I use the region_of_interest() functions, which requires the edge detected image and the vertices that the function should consider and leave while processing. I provide a quadrilateral vertices which is only considered for the lane detection. The function returns the edge detected image only inside the provided vertices. With this function we also call a function from the OpenCV library called cv2.fillPoly(), which is used to fill the pixels inside the polygon defined by "vertices" with the fill color. 

**Step 4:** Now comes the important steps to draw line on the detected edges of the lanes. We use Hough transformation for this purpose.  As we know the straight line equation 'y = mx + b', in the edge detected images, we consider each points in the (x,y) axis and convert to the parameter space(b,m). But with this process, there is a drawback. For a perpendicular line the slpe is infinity and hence the parameter space cannot be used. Hence comes the polar space 'rho = xcos(theta) + ysin(theta)' where, 'rho' is the distance from the origin to the closest point on the straight line, and 'theta' is the angle between the x axis and the line connecting the origin with that closest point.
Now we call the Hough transform function 'hough_lines()', which will call the HoughLinesP() function from the OpenCV2. 
"cv2.HoughLinesP(masked_edges, rho, theta, threshold, np.array([]), min_line_length, max_line_gap)"
In this case, we are operating on the image masked_edges (the output from Canny) and the output from 'HoughLinesP' will be lines, which will simply be an array containing the endpoints (x1, y1, x2, y2) of all line segments detected by the transform operation.
The threshold parameter specifies the minimum number of votes (intersections in a given grid cell) a candidate line needs to have to make it into the output. 'min_line_length' is the minimum length of a line (in pixels) that you will accept in the output, and 'max_line_gap' is the maximum distance (in pixels) between segments that you will allow to be connected into a single line.

Within the Hough_lines() function, we call another function called draw_lines(), which calculates the different cordinates in terms of (x1,y1), (x2,y2), between which the lines are drawn. 

The calculation follows like below:

the Hough_lines() function provides the values of x1,y1,x2,y2. But these values are scattered around the images and also changes during the iteration. For this purpose we have to calculate the slope and center points of those points and arrange it in an array. We also have to split the values based on the left and right line. If the slope is positive, then the values are sorted out for the left lane and for the negative slope the values are sorted out for the right lane. 
Then using those stored values we need interpolate and extrapolate the lines and find new values of x1,y1,x2,y2 to draw the lines by taking the average values of all the stored values of slope and (x,y) for right and left lane.

We assume the min and max values of y for both left and right lane and we use the slope to calculate the min and max values of x for both left and right lane.

min_yl = int(round(imshape[0]x(0.68)))
max_yl = int(round(imshape[0]))
min_yr = int(round(imshape[0]x(0.68)))
max_yr = int(round(imshape[0]))

The calculation for x is done in the functions 'right_lane_min' which provides the min x values for right lane, 'right_lane_max'which provides the max x values for right lane, 'left_lane_min' which provides the min x values for left lane, and 'left_lane_max'which provides the max x values for left lane.
min_xr = round(xr_center-((yr_center - min_yr)/avg_slopeR)).astype(int)
max_xr = round(xr_center-((yr_center - max_yr)/avg_slopeR)).astype(int)
min_xl = round(xl_center-((yl_center - min_yl)/avg_slopeL)).astype(int)
max_xl = round(xl_center-((yl_center - max_yl)/avg_slopeL)).astype(int)

with the new values of x1,y1,x2,y2, we proceed further to draw the line using "cv2.line(img, (min_xr,min_yr),(max_xr,max_yr), color, thickness)" on the blank image. 

**Step 5:** with the final values returned by the Hough function the function calls the "weighted_img(lines,image,α=0.8,β=1.0)" to draw the lines on the edge image. 

Hence, we finally succeeded to draw the required lines on the image. Now we proceed further to detect and draw line on the video provided. 

A video is a series of images, we use our above methods on all those series of images to detect the continuous lane lines and draw lines on them. We first call the video land brake them into images and call our defined function "process_image". The same procedure follows as described above to detect the lanes. It only requires some fine tuning in the paramets used inside the pipelines to work on different videos such as white lanes and yellow lanes. 
Now comes the challenge part, the challenge video is a bigger video with more number of pixels and is a curvy road. For this purposes, we need to increase the region of interest so that the lanes are detected accordingly. Also for the yellow lanes on a bright road is little tougher than imagined, but better results can be obtained with fine tuned parameters.

### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when the lanes are continuously changing and more curvy, then in those cases, the current pipeline would end up in the messy calculations of the x and y values. This would make the lines fluctuate more often. 

Another shortcoming could be when there are vehicles in front and the lanes are not visible to a larger extent. In this case the calculation of the lines would mess up and the vehicle might end up in following another lanes. Due to this we need more roboust solution. 


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to have more roboust solution in the calculation of the pipelines, specifically in those shortcomings mentioned above.  

Another potential improvement could be to have more fined tuning of the lanes when it comes to curvy lanes and avoid the fluctuation.
