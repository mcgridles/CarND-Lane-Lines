# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consists of 6 steps. First, I get a grayscaled image. Then, I apply Gaussian smoothing and run Canny
edge detection with a lower threshold of 50 and an upper threshold of 150. Next, I determine the region of interest
using a polygon which is parameterized using the image dimensions. Then, I calculate the Hough lines using a rho of 1,
theta of 1 radian, threshold of 15, minimum line length of 40, and maximum line gap of 30. Finally, I draw the lines
using my custom draw_lines() function.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by separating all of the
lines found in the Hough line step using their slope. All lines with a slope less than 0 belong to the left lane line, 
and all lines with a slope greater than 0 belong in the right lane line. Any lines with a slope that has an absolute 
value less than 0.5 or greater than 4 are skipped because they are most likely mis-characterized and will cause the line 
to jump around. I took the slope, and x and y values from the sorted lines and averaged them, then used those averages 
to calculate the y-intercept for each line. 

In some cases, all of the lines for a given lane line were filtered out because the dashes were too short to generate lines 
with slopes that had an absolute value greater than 0.5. Because of this, I kept a static variable that kept track of the last 
known slope, so if that ever happened then I could simply catch the divide by zero error and use the previous slope to 
calculate the y-intercept.

Finally, I used the slope and y-intercept to calculate the x-values at the edges (bottom and top) of the region of interest as 
defined by y-values. That gave me the two endpoints of each line which I then drew using the cv2.line() function.


### 2. Identify potential shortcomings with your current pipeline


There are two main shortcomings with my current pipeline. The first is that the lines still tend to be a little jittery.
I tried to reduce that by messing with the thresholds in the edge detection stage and the Hough line stage but I couldn't get
it to make much of a difference. I think it is mostly due to the fact that not all of the Hough lines have exactly the same
slope, and some even have a slope that is way off.

I tried to fix the jittering, and I think I did a pretty decent job, but it also led to my second shortcoming which is that it
is possible for all the lines associated with one of the lane lines to be filtered out. This causes a divide by zero error for
which I catch the exception and handle by keeping track of the last known slope, but it does mean that if it were to happen
for a prolonged period of time the calculated line could stray from the actual lane line. It would also not work on an image
since there would be no previous lane line from which to take the slope.


### 3. Suggest possible improvements to your pipeline

The biggest improvement would be to make my edge detection better. That way, I could still filter out the edges that cause my
lines to be jittery and still have edges left to calculate the slope and y-intercept with. This would most likely solve both
of my problems since it would allow me to filter a little more while still following the lane line.

In the big picture, it might be good to try to fit a polynomial line to the lane lines. That way, even if the road curves the
lane lines would still be accurate. This could be done by optimizing the standard deviation of the distance between the detected
lines and the line that is drawn on the image. However, that would be significantly more advanced I think.
