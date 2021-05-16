# **Finding Lane Lines on the Road** 

[//]: # (Image References)

[image1]: ./test_images_output/solidWhiteRight.jpg "Example line detection output"

## Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline is encapsulated in a single draw_lane_lines() function. It first coverted the imaged to grayscale and applies a Gaussian blur, then performs canny edge detection on the blurred image. Only then does it masl the image to blackout anything outside the region of interest (which is defined as an isoceles trapezium with upper base spanning the middle 1/8 of the image and the lower base equal to the x-dimension, and the height is equal to 2/5 of the y-dimentsion), since doing this before edge detection may cause the edge of the region of interest to be detected as edges. The edge-detected image then undergoes Hough transform to detect lines. 

At this point, quite a number of lines could be obtained. Instead of plotted all these lines individually, they can be consolidated into two groups: those on the left lane line, and those on the right. These two groups can be differentiated by calculating the gradient of each line: lines on the left  have negative gradients, whereas lines on the right  have positive gradients. The y-intercept can be calculated along the gradient quite easily. After grouping the lines, the average gradient and y-intercept of each group is calculated to obtain a final equation for the lane lines. Line segments which cover the lower 2/5 of the image are obatined from the line equations and plotted as the final lane line output, which is juxtaposed on the original image.

An example output looks like this: 
![Example line detection output][image1]

It did become clearer when this was applied to the video that the final lane lines can be skewed at times, presumably due to lines that are not the lane line itself but still picked up by Hough tansform nevertheless. Under the assumption that these outliers will have gradients and y-intercepts quite different from that of the actual lane line, we could calculate a slightly more sophisticated average (rather than a simple mean) to obtain the final gradient and y-intercept: we could calculate the mean of the values in the 2nd and 3rd quartiles, thuse filtering out some extereme values. When the lines are too few to form actual quartiles (i.e. less than 4 lines), we fall back to calculating the simple mean. This seems to offer some improvements against the skewed-ness of lines.

### 2. Shortcomings and possible improvements

A major issue with the current pipeline is that it does not gurantee that the lane lines can be successfully identified in each frame, as can be seen from the frames in the output video that where no lines are plotted. It can be argued, however, that this is not too serious a practical issue, as it occurs rather infrequently (that there are far more frames with lane lines detected than frame without), and such failures are generally transient. A simple practical solution is to use the most recent lane lines when one is not detected.

Another issue is that since the lane markings have 2 edges, either one of them could be identified as a line (if not both). This causes the detected line to frenquently shift between the left and right edges, which could be a problem in practical scenarios. A possible way to address this to increase the max_line_gap setting used by Hough transform, which may ultimately ensure that the two edges are detected as separate lines. From there, we could further analyse the equations for each line segment to always pick the inner edges. 

The pipeline is also ineffective when sharp curves are encountered. This is expected as its operation relies on the premise that the lines detected in the region of interest do, in fact, lie on a straight line. This is obviously not the case in sharper curves. Without specifically coming up with a method for curve detection, a possible way to address this is to have a smaller region of interest -- at the peril that the lines detected may be less robust even in the straight line cases. Another possible strategy is to add a weight to the line segments when computing the average, such the line segments closer to the bottom are given significantly higher weights. Doing so allows line segments farther away from the bottom to still contribute to the overall average without segments on the curve (which tend to be farther away from the bottom) having too huge an impact on the output. 
