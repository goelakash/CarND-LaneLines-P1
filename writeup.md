# **Finding Lane Lines on the Road** 

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/solidWhiteCurve_withLanes.jpg "Lane"

---

## Reflection

### 1. Description

My pipeline consisted of X steps. 

1. Convert the image to grayscale.
2. Blur the image using GaussianBlur. I used a filter size of 11 based on my experimentation with the GUI tool for Canny edge detection here: https://github.com/maunesh/opencv-gui-helper-tool
3. Detect the edges using Canny edge detection. Used a low and high threshold of 40 and 150 respectively, which is close to the 1:3 ratio recommended for the algorithm.
4. Calculate the vertices for the region-of-interest. I use a trapeziodal area, with the bottom equal to the image width and top equal to 1/25th of the width (to simulate lane shape).
The top edge is shifted by 50 pixels downward to avoid the noise in edge detection in the centre of image, which comes from cars entering the lane apex. 
5. Generate the region of interest using the vertices on the detected edges and get rid of unnecessary edges outside lane-area.
6. Detect line-segments using HoughTransformP and select the longest two line-segments with opposing slopes<sup>*</sup>.  
The line-segments were extrapolated in both directions to find:
a) Their point of intersection
b) Where they meet the bottom of the image
These extrapolated points were used as the end-points for drawing the lanes.
7. Cache lanes from previous image. For video, which sometimes has a few frames without proper lanes, I added a `prev_lanes` argument to the `draw_lines` method, which can be used to draw lanes using last computed lane co-ordinates in case the current image/frame did not have enough edge information in the lane-related region-of-interest.


![alt text][image1]


### 2. Shortcomings:


1. Currently I rely on line-segments from both sides without checking for duplicate lanes on either side. That means that if the car was on a middle lane, and there were two or more lane markers on eother side, its possible that if the nearest lane line is not visibile due to some reason, then the pipeline may take the next lane as the correct lane. This means that it will include the adjacent lane as part of current lane.
2. I don't make the distinctintion b/w left and right lanes. It is possible that within a frame I only get edges for either of the lanes, but not both. I've added a cache to prevent this situation, but this needs to be handled better.


### 3. Possible improvements to your pipeline

 1. The cache with last lane coordinates currently works if both the lane lines were present at some point. But if a curve comes, and the lane lines start interchanging (e.g. left lane is visible in current frame but right frame is not, but after a few frames its vice-versa), then we will continue to use old cached values and the system will not get updates in changing lane directinos. We can fix that by changing the code to cache any lane that is visible in the frame.
