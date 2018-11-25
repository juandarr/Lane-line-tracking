# **Finding Lane Lines on the Road** 

## Writeup to explain the processes followed in the completion of this project

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images/solidWhiteCurve_lanes.png "Lane detection 1"
[image2]: ./test_images/solidWhiteRight_lanes.png "Lane detection 2"
[image3]: ./test_images/solidYellowCurve_lanes.png "Lane detection 3"
[image4]: ./test_images/solidYellowCurve2_lanes.png "Lane detection 4"
[image5]: ./test_images/solidYellowLeft_lanes.png "Lane detection 5"
[image6]: ./test_images/whiteCarLaneSwitch_lanes.png "Lane detection 6"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. First, I converted the images to grayscale, then I created a smoother image using Gaussian Blur with a square size of 3 pixels. The third step applies the 
Canny Edge Detection algorithm to identify edges in the image based on the gradients of intensity in the pixelated image. To detect just the edges belonging to the lines a region of
interest is defined in step 4, masking the edges image with the region of interest. This region is a 4 sided polygon and the idea is that
it will only include the lane limits of the current lane followed by the car. Once the edges have been identified and masked to the region of interest, step 5 uses the hough transform to detect straigh lines joining pixels in those edges. Given the parameter definition rho = 1,
    theta = np.pi / 180,
    threshold = 15
    min_line_length = 5 and 
    max_line_gap = 3, we expect to identify lines joining 5 or more points, with a rho of 1 or more pixels, every np.pi / 180 radians, with a minimum line length of 12 pixels and maximum gap between lines of 3 pixels. After indentifying the lines, a modified version of draw_lines function calculates
which lines belong to the left or right lanes and applies least squares polynomial fit to calculate the slope and y axis intercept of a unique line joining the set of lines in each group (left or right lines).
Finally in step 6, we combine the original image and the resulting lanes (in red). This gives us
an indication of what the machine sees and identify as lane limits during different video tests.

#### Explanation of improved version of draw_lines(). Use argument mode=1 to call this option. The default mode value (mode=0) will call the initial version.

The draw_lines() function was modified to group the lines in two sets: left lane if the slope of the line is negative, and right lane if the slope is positive. Additionally only appropiate slopes were chosen filtering those lines that are too close to the horizontal or vertical lines.
Once the two groups of lines have been defined (through the addition of their endpoints), least squares is used to define a unique lane in each group. Finally each line is drawn and overlaped on the original image.

The following images show the lane detection pipeline on the job. It has been applied to every sample image provided in the repository:

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]


### 2. Identify potential shortcomings with your current pipeline

There are several considerations that have been done in the project that won't necessarily be met in real world scenarios. 
- One of them is that the lines detected inside the region of interest are expected to belong only to the right or left lanes. This is not a realistic expectation as seen in the challenge video where changes in the material of the road and shadows create a more
complex global picture inside the four sided polygon. Even in the two test videos there are some irregularities in the road (most of them cracks) which are detected by the Canny Edge detection algorithm, and some set of points translate in straight lines depicting those cracks after hough transformation. This is not what we want since we are only interested in the lanes.

- The four sided polygon is also expected to depict this region of interest in an accurate way all the time, this won't be the case when the position/angle of the camera changes or the car is changing tracks. 

### 3. Suggest possible improvements to your pipeline

- We need a region of interest able to detect transitions in the position/angle of the sensor and adapt to it. 

- Also, for different frames of reference (for example if the road is narrow, wide, short,long, etc.) the shape of the four sided polygon needed to accurately depict the region of interest will be different (a different trapezoidal shape should be required). 

- Even if we achieve this milestone, what happends when the car is changing of lanes? In this scenario, using the same current algorithm the region of interest will identify as region of interest a region that is not expected (middle of the lane). The addition of two or more region of interest could help us to clearly defined that the road has multiple lanes and that we are in a transition.

### 4. Challenge

There are several aspects relative to the challenge video that results in the pipeline algorithm performing poorly:

1. There is a constant curve on the road. This makes detection of straight lines as right or left lane more difficult since this is not their natural shape (in curves, lane lines appear as curves). One potential solution here is to shorten (vertically) the region of interest so we can focus in lane detection for regions of interest where the lanes are likely straight lines or make a more robust algorithm where lanes can be curves.
- Implementation: Vertically shorten the region of interest for lane detection where lanes are straight lines. 
2. During the period of time between the fourth  and sixth second there is a change of terrain and shadows from trees appear over the dark terrain. The terrain changes from dark to bright (during second four), and eventually from bright to dark (during second six). The canny edge detection algorithm detects undesired edges during the changes of terrain, the shadows from the trees and areas of dark material over the bright terrain. Also the yellow lane is more difficult to detect over bright terrain. Light intensity has a clear impact in the algorithm. This observation is widely known in computer vision, where light exposure has a relevant influence in the performance of algorithms.
- Implementation: create/use an algorithm to detect and ignore shadows and changes of terrain (asfalt in this case) on the road. Make the algorithm more robust during changes of light exposure.
3. There is some weird brown liquid/solid area at the bottom of the video (probably the front of the car interior). The Canny edge algorithm detects edges in the transitions of pixel intensity, which have consequences in posterior processes such as the Hough transform and the definition of the lanes.
- Implementation: Remove this undesired area from the region of interest since it does not add any value to the goal of lane detection. 

The proposed implementations (1) and (3) were applied to the challenge. (2) will be applied in future developments. 