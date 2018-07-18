# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

Images save in `test_images_output/`

Videos save in `test_videos_output/`


[//]: # (Image References)

[image1]: ./test_images/solidWhiteCurve.jpg "Grayscale"
[image2]: ./test_images_canny/solidWhiteCurve.jpg "Canny"
[image3]: ./test_images_hough/solidWhiteCurve.jpg "Hough"
[image4]: ./test_images_advance/solidWhiteCurve.jpg 
[image5]: ./test_images_output/solidWhiteCurve.jpg 
[image6]: ./test_images_naive/solidWhiteCurve.jpg 

---

## Pipeline
### 1. use original help function
First of all, I converted the images **(take the solidWhiteCurve.jpg for example)** to grayscale, then used canny algorithm to dectected edges in the image.

![alt text][image1]
![alt text][image2]

Canny edge detection apply in  whole picture, but we are just interested in the samll region which lane lines are more likely to appear in front of the camera. Before implemented a Hough Transform on edge detected image, we selected an interest region. The parameters in this processs are below:

```
    #interest region
    vertices = np.array([[(150,image.shape[0]),(420, 330), (570, 330), (900,image.shape[0])]], dtype=np.int32)
    
    #Define the Hough transform parameters
    rho = 2 # distance resolution in pixels of the Hough grid
    theta = np.pi/180 # angular resolution in radians of the Hough grid
    threshold = 15     # minimum number of votes (intersections in Hough grid cell)
    min_line_length = 40 #minimum number of pixels making up a line
    max_line_gap = 20    # maximum gap in pixels between connectable line segments
    
```
![alt text][image3]

Then used help_function `weighted_img()` to combine the hough transform output and original image.

![alt text][image6]

Here is the code of the lane line detected pipeline:
```
def process_image(image):

    gray = grayscale(image)
    kernel_size = 5
    blur_gray=gaussian_blur(image, kernel_size)
    low_threshold = 50
    high_threshold = 150
    #the best ratio is 1：3
    edges = canny(image, low_threshold, high_threshold)
    
    # define masked region 
    vertices = np.array([[(0,image.shape[0]),(450, 330), (490, 330), (image.shape[1],image.shape[0])]], dtype=np.int32)
    masked_edges=region_of_interest(edges, vertices)
   
    # Define the Hough transform parameters
    rho = 2 # distance resolution in pixels of the Hough grid
    theta = np.pi/180 # angular resolution in radians of the Hough grid
    threshold = 15     # minimum number of votes (intersections in Hough grid cell)
    min_line_length = 40 #minimum number of pixels making up a line
    max_line_gap = 20    # maximum gap in pixels between connectable line segments

    line_image=hough_lines_advance(masked_edges, rho, theta, threshold, min_line_length, max_line_gap)
   
    result=weighted_img(line_image, image, α=0.8, β=1., γ=0.)
   
    return result
```

### 2. modify draw_lines() function
As you can see, the helper function just detected the segmented lines. If we want to solid lines rather than segments, we have to modify the function `draw_lines()`.

Here is the first version code, just extended every single line in the region of interested.
```
def draw_lines_advance(img, lines, color=[255, 0, 0], thickness=3):

    for line in lines:
        for x1,y1,x2,y2 in line:
            k=(y2-y1)/(x2-x1)
            if -2<k<-0.5 or 0.5<k<2:
                ave_x=(x1+x2)/2
                ave_y=(y1+y2)/2
                y1=330
                y2=img.shape[0]
                x1=(y1-ave_y)/k+ave_x
                x2=(y2-ave_y)/k+ave_x
                cv2.line(img, (int(x1), y1), (int(x2), y2), color, thickness)
```
![alt text][image4]

Based on the 1st version code, I want to draw  one line on each sides. 

The solpe define by `k=(y2-y1)/(x2-x1)`. 

If `-2<k<-0.5`, the line belong to the left sides.

If `0.5<k<2`, the line belong to the right sides.

In this way, find the center of the lines and averege the slope, then draw the solid line.

```
def draw_lines_challenge(img, lines, color=[255, 0, 0], thickness=10):
    x_l=0
    x_r=0
    y_l=0
    y_r=0
    k_l=0
    k_r=0
    n_l=0
    n_r=0
    #initial parameters
    for line in lines:
        for x1,y1,x2,y2 in line:
            k=(y2-y1)/(x2-x1)
            if -2<k<-0.5 :
                x_l+=(x1+x2)
                y_l+=(y1+y2)
                k_l+=k
                n_l+=1
            elif 0.5<k<2:
                x_r+=(x1+x2)
                y_r+=(y1+y2)
                k_r+=k
                n_r+=1
    #avoid division by zero
    if n_l!=0:
        k_l/=n_l
        x_l/=(2*n_l)
        y_l/=(2*n_l)
        y1=330
        y2=img.shape[0]
        x1=(y1-y_l)/k_l+x_l
        x2=(y2-y_l)/k_l+x_l
        cv2.line(img, (int(x1), y1), (int(x2), y2), color, thickness)
    if n_r!=0:
        k_r/=n_r
        x_r/=(2*n_r)
        y_r/=(2*n_r)
        y1=330
        y2=img.shape[0]
        x1=(y1-y_r)/k_r+x_r
        x2=(y2-y_r)/k_r+x_r
        cv2.line(img, (int(x1), y1), (int(x2), y2), color, thickness)
```
![alt text][image5]




## Shortcomings 

One potential shortcoming is that the algorithm sometimes can not find the lane lines when process the video. This mean that algorithm will division by zero which is a special case.

Another shortcoming could be that the algorithm can not deal the suitation when a car is close to the camera. There is no doubt that the algorithm would detecte the car as lane line.


## Possible improvements 

I used to focus on C/C++, so I am not familiar with the python library. The code can be more compacter by using python library.
