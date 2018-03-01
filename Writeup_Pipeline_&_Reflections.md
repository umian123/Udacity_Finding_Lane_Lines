# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: Images: Please see the notebook where I have separately printed images for the 6 images after each processing step, so total of 36 images. Thanks!

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

Please see the notebook where I have separately printed images for the 6 images after each processing step. Thanks!

My pipeline consisted of 5 steps:

Step 1: Converted given test images into grayscale. Grayscaling is done because the next step (Canny edge detection) algorithm takes into account the changes in pixel strength (or gradient) to detect the boundaries of different entities in the image, and pixel strength/intensity in a grayscaled image is shown as how bright it is. Printed resulting images in this and subsequent steps too.

Step 2: Applied Gaussian smoothing on the grayscaled image. Gaussian smoothing is probably not super important when the edges/lines are not rough or already smooth, but since in real world we'll encounter many rough edges so smoothing will be applied to filter out noisy edges due to rapid pixel gradients. Set the Kernel size as 5.

Step 3: Defined and tuned Canny edge detection thresholds to be applied on the above grayscaled image. Resulting images had their boundaries detected by the algorithm, in preparation for drawing lines on them in next steps (Canny edge image represented a collection of 'dots' or points in image space, which would then be transformed to intersecting lines during Hough transform step. Set high threshold value such that only the changes in pixel intensity (pixel gradients) with values higher than that were accepted, while gradients lower than low threshold were rejected. This is another aspect of filtering, where we want to fine-tune the parameters so that too high threshold doesn't reject most of the necessary lines, and too low of the high threshold doesn't accept too many edges. Had to tune this extensively in order for the Challenge video to look much better than original.

Step 4: Defined a four-sided polygon as region of our interest in the images, just to encapsulate the boundaries of two regular road lane lines, going from bottom of the image, wider apart, and narrowing in as the lanes extend till an area a little bit below the center point in the image frame.

Step 5: Applied Hough transform which drew red lines on the defined polygon (region of our interest) and masked everything else in the images as dark/black.
     5a: Draw_lines_RAW: Applied just the raw Hough transform first on the solid white video, in order to show rough line segments drawn on the video to fulfill the rubric item similar to raw-lines-example.mp4. The output just drew detected rough line segments, annotating left and right lines/line segments.
     5b: Draw_Lines_Solid: (to go from raw video to solid lines) Since there are multiple lines detected for a lane line (recalling that multiple points/pixels in image space corresponds to multiple intersecting lines), but we are only interested in a composite/average and clearly annotated line on one side and one on the other (left and right lanes). So the main 'draw_lines' function was applied to split the various intersecting lines by slope.
     - Splitting: So first line splitting was done using the well-known line slope formula. For image space (unlike cartesian space), left lines have negative slopes and right lines have positive slopes. So I split the left and right lanes using slopes. For the challenge video, I had to go back and filter out further noisy lines by tuning how both side of lines were split using slope (e.g. instead of saying that if line_slope > 0 then it is right line, I said 'if line_slope > 0.3 then it is right line' and similar for left lines, to even distinguish/split right and left lines with wider margin around 0. Also calculated the intercepts of these split lines as a line is defined by slope and intercept.
     - Averaging & Extrapolating: Since there were still multiple split lines, I averaged the split lines since we only want one left and one right line drawn. So averaged line slopes and intercepts separately using np.average function, and the the x_points (x1, x2 - since we need these and y1, y2 for the cv2.line function to mutate original image with our drawn lines) were calculated by using (y = mx + c --> x = (y-c)/m), where c and m were averaged single intercept and slope, respectively. This final step also extrapolates the lines based on partially detected line segments, so that full extent of image frame is covered.
     - Finally, using cv2.line function, the original un-processed (or edge) image was mutated by drawing solid weighted lines on it with minimal weighing factors.


Some images to show how the pipeline works: 
Please see the notebook where I printed each given image after each processing step, thanks.




### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming I'd like to mention is from the 'Challenge' video, where on the bridge where the normal road area changes color due to changed material, it seems that the left yellow line becomes very dim and hard to recognize and so the line falters in that area. It seems that in that video, white line is recognized better than yellow, so better image color spcae conversions probably need to be applied instead of simple RGB to grayscale and back.

Another shortcoming is that so far the lines are only straight and simple, and we are not drawing on advanced curved roads.

Yet another shortcoming is that lines are not moving as smoothly in the video as I would like them to.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to employ better color space processing/conversions instead of simple RGB since in the challenge video, the pipeline didn't detect some not so bright yellow lane patches.

Another potential improvement could be to better scale the averaging of the lines process, where possibly longer/stronger lines would be given more weight than weaker ones, to reduce the noise in the Challenge video.
