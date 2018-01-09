# **Finding Lane Lines on the Road** 

## Writeup
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # "Image References"

[image0]: ./pipeline_steps_images/0_input.png "Input"
[image1]: ./pipeline_steps_images/1_gray.png "Grayscale"
[image2]: ./pipeline_steps_images/2_blur.png "Blurred"
[image3]: ./pipeline_steps_images/3_cany.png "Cany"
[image4]: ./pipeline_steps_images/4_masked.png "Masked"
[image5]: ./pipeline_steps_images/5_hough.png "Hough"
[image6]: ./pipeline_steps_images/6_lines_left_right.png "Sorted"
[image7]: ./pipeline_steps_images/7_single_lines.png "Single lines"
[image8]: ./pipeline_steps_images/8_result.png "Result"

---

### Reflection

### 1. The Pipeline 

The pipeline used in this project to detect the left and right lane markers on the lane currently driven reuses most of the helper functions given in the P1 notepad. The `hough_lines()` function has been extended to return the lines additionally to the image it original did.

Additional helper functions have been declared to seperate lines for the left and right side (`order_lines()`) and to average the lines parameters (`average_lineparams()`). The main processing function (`process()`) consists of the following steps;

**Steps**

1.  **Input / Set variables and parameters**

   Though not a processing step, first all parameters are set in a central place and image sizes are stored conveniently. Additionally there are switches to output debug values and to save intermediate images as shown in this document.
   ![alt text][image0]

2. **Greyscale**

   the input image / frame is converted to greyscale
   ![alt text][image1]

3. **Blur**

   To make the image a bit smoother for contrast / canny detection  it is blurred to sort out single pixels with a bluring kernel of $3$
   ![alt text][image2]

4. **Cany edge detection**

   To find edges, the cany algorothm is used with a lower and upper threshold of $50$ and $150$
   ![alt text][image3]

5. **Region of interest selection**

   From the rectangular image a region of interest in form of a trapezoid is selected. The trapezoid, at the bottom, has the width of the image (`mx_bottom=0`), at the top, at an height of $40$% from the bottom, the width is $10$% of the image. The region is set in relative values to account for different sized images/frames as needed for video 3.
   ![alt text][image4]

6. **Hough transformation / line detection**

   On the selected image region the lines are detected with the Hough algorithm with $\rho = 2$, $\theta = \pi/180$, $threshold=11$, $minLineLength=20$ and $maxLineGap=3$

   Additional to the image the points of the lines are returned as well by extending the return code of `hough_lines()`to

   `return [line_img, lines]`

   ![alt text][image5]

7. **Ordering and Filter**

   To select which lines belong to the left *(green)* and the right *(red)* side, I created the function `order_lines()`. Lines whose middle points lay in the left half of the image are considered for the left lane border, those whose middle points lay in the right half for the right one.

   To filter out lines with an unwanted slope, the lines' slopes are defined as $m=\Delta x / \Delta y$ , the complete line is represented as $x=my+b$. The coordinate-system-switch from the more usual $y=f(x)=mx+b$ has been made to allow for perpendicular lines in the image which otherwise would have a slope of $m=\pm \infty$ .

   In the given representation, lines with an absolute slope larger than $2$ are discarded. This is especially helpfull for the 3rd video.

   ![alt text][image6]

8. **Averaging lines**

   For each side in the function `average_lineparams()` the line's parameters $m$ and $b$ are combined weighted by the square of the line's length to lower the influence of smaller lines that have a higher likelyhood of having another slope than the lane border marking. The weighting has been added before the filter was added in the previous step and may therefore not be of too great value in the end result but is kept in anyway.

   With these averaged parameters a line for each side is then drawn within the region of interest.

   ![alt text][image7]

   ​

9. **Result**

   Last, the lines are combined with the original image and returned as result.

   ![alt text][image8]

   ​



### 2. Potential shortcomings with the current pipeline

The current implementation may have the following shortcomings / problems:

- **Region of interest**

  The Region of interest is selected statically, this may cause problems when close to a lane border, as relevant parts may be cut out. Furthermore there might be problems with structures above an uneven road.

  Lines out of the current lane are ignored which might be usefull to determine where on the road we are

- **Straight lines**

  The pipeline uses straight lines which may cause problems in close road curves(/turns?)

- **Slope filtering**

  Filtering the slope may cause problems when in close road curves(/turns?), as well as ignoring lane markings from crossing roads or for stop-lines before crossings

- **No color filtering**

  The current pipeline ignores the color of elements and only works on contrast and would therefore happily accept e.g. blue or green lines as road markings as well. Furthermore, at least in Europe, yellow lines often denote a higher priority and are used above normal white lines in construction zones for example. This is totally ignored for now.



### 3. Possible improvements

In line with the potential problems stated in chapter 2, the pipeline could probably be improved by using a **color filter** to allow only yellow and white elements to be considered for lines. Furthermore (projected) **clothoids** could represent the line markings in a more accurate way, espacially in curves(/turns?). By considering time and more than one frame at a time, **a filter over time** could be implemented to account for short problems in line detection and single 'outlier' frames