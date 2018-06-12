# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[solidWhiteRight]: ./test_images_output/solidWhiteRight.jpg "solidWhiteRight.jpg"

[shortcoming1]: ./shortcomings/shortcoming1.png "shortcoming1"

[shortcoming2]: ./shortcomings/shortcoming2.png "shortcoming2"

[shortcoming3]: ./shortcomings/shortcoming3.png "shortcoming3"

---

## Reflection

### 1. Pipeline
Here are the steps in the pipeline:
1. Convert to Grayscale
2. Apply Gaussian Blur
3. Perform Canny Edge Detection
4. Apply Region of Interest Mask
5. Draw Lines using Hough Transform

The rationale behind the ordering of steps is as follows -
- I need to run Canny Edge detection to get edges in the image.
- The color inside these edges do not matter, hence a grayscale version of the image was used.
- To avoid detecting too many edges, a Gaussian blur was applied on the image. 
- Lane lines are significantly thick in the image so even with a Gaussian blur, their edges will not be lost.
- However, the blurring reduces the number of noisy edges that we detect using Canny Edge Detection.
- Unwanted edges can further still be discarded by applying a region of interest mask.
- The ROI mask was applied after Canny Edge Detection to avoid data loss. 
- If the ROI mask was applied before Canny Edge detection, it is possible that certain important gradient changes at the border of the region of interest are not detected correctly.
- Hough lines are drawn on the edges that remain after applying the ROI mask.
- The result is as such -

![alt text][solidWhiteRight]

#### draw_lines() was developed iteratively in the following way:

### I

Initial verison of **draw_lines()** was simple. It separated the Hough lines into two sets - one had lines with positive slopes and one had lines with negative slopes. Lines with positive slopes were part of the left lane and lines with negative slopes were part of the right.

The average slope for each set was calculated by averaging the slopes of all the lines in the set. 

*average_slope = (slope1 + slope2 + . . . + slope_n) / n*

The average position for each set was calculated by averaging the *x* and *y* coordinates of both the points in each of the lines in the set.

> *average_x = ((x11 + x12) + (x21 + x22) + . . . + (xn1 + xn2)) / (n\*2)*

> *average_y = ((y11 + y12) + (y21 + y22) + . . . + (yn1 + yn2)) / (n\*2)*

> *average_position = (average_x, average_y)

With the average slope and average position, the equation of the line representing the lanes was found using the formula *y = mx + c* .

Using the above equation, the length of the line was extrapolated to the top of the region of interest and the bottom of the frame.

This method of determining the lines produced lines that were often not aligned with the lane. This was because there were Hough lines in the sets that were almost horizontal or vertical which shifted the average slope or position.

### II

In the next iteration of **draw_lines()**, Hough lines with slopes in a range were grouped together. This is achieved using the **group_lines_by_slopes()** function. 

**group_lines_by_slopes()** puts lines whose slopes vary by an absolute percentage threshold into the same group. Each time a line is added to the group, the average slope of the group is updated.

**group_lines_by_slopes()** is used to get the groups of slopes for each set of lines. The group with the maximum number of lines in each set was then used to calculate the average slope and position of the lanes.

For example:


```python
# right_lines - Lines with negative slopes
# Groups of negative slope lines
right_lines_by_slopes = group_lines_by_slope(right_lines) 

# Get slope with maximum number of lines
right_max_key = max(right_lines_by_slopes, key=lambda s: len(right_lines_by_slopes[s]))

# Get average slope and position of the group with the maximum number of lines
right_average_slope, right_average_position = get_average_slope_and_position(right_lines_by_slopes[right_max_key], shape=imshape)
```



```python
# left_lines - Lines with positive slopes
# Groups of positive slope lines
left_lines_by_slopes = group_lines_by_slope(left_lines) 

# Get slope with maximum number of lines
left_max_key = max(left_lines_by_slopes, key=lambda s: len(left_lines_by_slopes[s]))

# Get average slope and position of the group with the maximum number of lines
left_average_slope, left_average_position = get_average_slope_and_position(left_lines_by_slopes[left_max_key], shape=imshape)
```

### III

In the third iteration, instead of taking averages of the Hough lines as is, the Hough lines were extrapolated to their endpoints at the top of the region of interest and at the bottom of the frame; the position of these extrapolated lines were averaged to get the average position of the endpoints. The slope of the line connecting the averaged endpoints was used as the average slope of the lane. And the center of this line was used as the average position of the lines.


### 2. Shortcomings

Here are two shortcomings of the algorithm -

1. The slope calculated for the lanes doesn't account for the thickness of the lanes. This results in lines which don't correctly align with the lanes. It results in lines like these -

![alt text][shortcoming1]

In the image above, although the red line on the right is drawn over the solid white line, its slope isn't the same as the white line. The same shortcoming is seen in the red line on the left. This miscalculation in slope causes the two red lines to intersect at the top.

![alt text][shortcoming2]

In the image above, this shortcoming is more clearly seen in the red line on the left. The miscalculation of the slope causes the red line to be extrapolated much futher away than expected.

2. When separting the Hough lines into two sets containing positive and negative sloped lines, I don't account for the position of the lines in the image. As a consequence, some noisy lines on either side ends up changing the average slope and position of the opposite line. It results in lines like this - 

![alt text][shortcoming3]

In the image above, the top end point of the red line on the right is beyond the expected end point of the red line on the left. This is probably because the average slope of the right lines were affected by a few noisy lines detected from the left lane.

### 3. Improvements

1. When dealing with videos, a global average slope and position for both the lines can be maintained. This average can be updated each time a new frame is processed. This algorithm is short sighted currently in that it calculates the average slope and position on in the current frame. Maintaining a global average would filter out noise further. It frames similar to the second shortcoming image, a global average would help get a better average for the left line.

2. This algorithm performed poorly on the **challenge.mpg4** video. An improvement to handle frames with changing color patterns on the road due to shadows or asphalt would be to account for the color of the lines that we're tracking. A color mask to ignore all colors outside the expect thresholds would leave an image with mostly just the lanes. Applying this algorithm on such an image would render better lines.
