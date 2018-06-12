# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### Pipeline
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
- If the ROI mask was applied before Canny Edge detection, it is possible that certain important gradient changes are the border of the region of interest are not detected correctly.
- Hough lines are drawn on the edges that remain after applying the ROI mask.
- The result is as such -
![solidWhiteRight.jpg][test_images_output/solidWhiteRight.jpg]

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
