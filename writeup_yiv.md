# **Finding Lane Lines on the Road** 
# **by Y.Ivanov**
   
## Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. My pipeline description.

My pipeline consisted of 10 steps:
1. Set region of interests to from 0.6 to 0.92 of heigth and form 0.1 to 0.9 of width of original image, also i've extended top of region of interests by 40 pixels: 

``
    w = in_image.shape[1]
    h = in_image.shape[0]
    masked_image = region_of_interest( in_image, np.array([[ (w*0.1, h*0.92),
                                (w*0.5-40, h*ROI_FACTOR),
                                (w*0.5+40, h*ROI_FACTOR),
                                (w*0.9, h*0.92),
                                ]], dtype=np.int32) )

``

![image masked by region of interest][writeup_yiv_images/01_masked_by_region_of_interest.png]

1. Invert image, because according to my expirince it able easyly detect blue and black color in some corner cases when yellow and white:

``
    inverted_masked_image = cv2.bitwise_not( masked_image )
``

![inverted image][writeup_yiv_images/02_inverted_masked_image.png]

1. Mask image by blue and black colors and grayscale image: 

``
    mask_blue  = cv2.inRange(inverted_masked_image, (0, 0, 128), (60, 110, 255))
    mask_black = cv2.inRange(inverted_masked_image, (0, 0, 0), (45, 45, 45))
    mask = cv2.bitwise_or(mask_blue, mask_black)    
    ranged_image = cv2.bitwise_and(masked_image, masked_image, mask=mask)
    gs_image = grayscale( ranged_image )
``

![ranged image][writeup_yiv_images/03_ranger_by_color_masks.png]

1. Apply Gaussian blur with kernel size 6:

``
   blured_image = gaussian_blur( gs_image, 5 )
``

![blured image][writeup_yiv_images/04_blured_image.png]

1. Apply Canny edge detection beetwin 50 and 90:

``
   canny_image = canny( blured_image, 50, 90 )
``

![canny image][writeup_yiv_images/05_canny_image.png]

1. Apply Hough line transform: 

``
    lines = cv2.HoughLinesP( canny_image, 1, math.pi/180, 
                            threshold = hough_treshold, 
                            lines=np.array([]), minLineLength=5, maxLineGap=300)
``

``
slope = -0.579, length=249.562, bottom_x= 154.776, top_x=615.115 [255 662 471 537]
slope = -0.672, length=224.100, bottom_x= 189.696, top_x=586.099 [276 662 462 537]
slope = -0.623, length=240.302, bottom_x= 171.835, top_x=599.753 [265 662 469 535]
slope = -0.621, length=238.925, bottom_x= 168.556, top_x=597.756 [262 662 465 536]
slope = -0.677, length=194.427, bottom_x= 188.807, top_x=582.297 [273 663 434 554]
slope = -0.614, length= 97.417, bottom_x= 156.725, top_x=590.278 [256 659 339 608]
slope = -0.619, length= 98.793, bottom_x= 159.077, top_x=589.415 [256 660 340 608]
slope = -0.622, length=236.698, bottom_x= 166.344, top_x=594.715 [258 663 459 538]
slope = +0.675, length=196.644, bottom_x=1108.382, top_x=713.625 [732 466 895 576]
slope = +0.600, length=128.281, bottom_x=1140.000, top_x=696.000 [790 510 900 576]
slope = -0.626, length=239.454, bottom_x= 174.291, top_x=600.112 [267 662 470 535]
slope = -0.648, length=233.549, bottom_x= 181.488, top_x=592.625 [271 662 467 535]
slope = -0.580, length=239.268, bottom_x= 156.950, top_x=616.490 [257 662 464 542]
``

1. Iterate trough detected lines:
   1. Detecte a slope, length and intersection of lines. 
   1. Accumulate intersections for left and right lines weighted by detected line length
   1. calculate intersection points
1. Applay hough_lines function:

``
        hough_image = hough_lines( canny_image, 1, math.pi/180, hough_treshold, 5, 300 )
``

![hough image][writeup_yiv_images/06_hough_image.png]

1. Draw weithed lines with thikness 5:

``
   left_bottom_x=170.749, left_top_x=598.092,right_top_x==706.667, right_bottom_x==1120.865 
``

``
   draw_lines( original_image, [ 
       [ [ int( xbl/l_len ), int( h ), int( xtl/l_len ), int(h*TOP_FACTOR) ], 
         [ int( xbr/r_len ), int( h ), int( xtr/r_len ), int(h*TOP_FACTOR)            ]
       ] ], thickness=5 )
``

1. And recive somethig like this:

![result][writeup_yiv_images/09_result_image.png]

### 2. Identify potential shortcomings with your current pipeline

Weather, lighting or road surface conditions can broke the detection algorithm. Actually, looks like implemented pipeline bruteforced to work only with provided samples.   

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to use several channes detection in parallel 

Another potential improvement could be to use filters or moderm ML/DL algorithms. 
