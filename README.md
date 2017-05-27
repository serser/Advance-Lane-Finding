# Advance-Lane-Finding
Implementing advance lane finding using computer vision techniques

Results Video can be found here: https://www.youtube.com/watch?v=KeMb0zTK9yA

## Introduction and Outline to the project

For this project the goal was to take camera images from a car, undistort the images based on the camera's calculated calibration settings, retrieve only pixel values of interest in a binary image. Then perform a perspective transform on the binary image and use a very robust lane tracking algorithm to find lane curve positions. Finally all the results are graphically tied together in outputing the orginal image with augmented graphics overlays of the measured lane lines.

There were two very important steps necessary for getting good lane position results that were stable and accurate. The first was using x/y gradient and S/V color channel thresholding to get a binary image. Finding the right thresholding values was anything but trival and required alot of experimentation to get right. Whats more is it turned out thresholding values that worked really well for one video did not work well at all for other videos, this meant that tyring to make a system that was general and dynamic for finding good pixel values was difficult. There was actually a lot of expeirmentation done trying to dynamically found good thresholding values from exposure measurments, the idea was if the image had too little bright pixels to increase the thresholding until the right percent was found. Likewise if the image had too many pixels to raise the threshold. This technique showed some promise but, it was thought best to instead focus on creating a very good line class tracker instead, which the project emphasised so preset thresholding values were picked depending on the video. Ideas in the future for producing general binary images could be to even use machine learning to teach a CNN type network to output pixels of importance, almost could be bulky enough to base a graduate research project on perhaps. Heres a great paper that could inspire the work, http://news.engineering.utoronto.ca/new-ai-algorithm-taught-humans-learns-beyond-training/

The second important step then was the line tracking class, which the project suggested was the highest importance. During the lecture videos it was first talked about using seperate left/right window sliding techniques to find lane lines going from bottom to top. This was a great idea for a starting point and it worked rather well on the first project video without even saving past frame values (if the binary image was good). On the challenge video however, a line class need to be implemented that did as the project suggested and stored line positions, averaging over time and only considering new curves from past curves, but also trying to understand when to start over and find a new base line. The line class worked to get fairly good results for the challenge video, but then the final harder_challenge video was attempted.

The line class uterly failed at trying to grasp the complexity of the final video, fist of all it was very difficult to get a clean binary image, there was still basic form but a lot of noise. The basic window sliding tracker could not handle this noise and would easily get misdirected from both sides, leaving lane markings that were not even parallel let alone accurate. Next the line class tried average results and ignore results it found bad, this meant the class had a very hard time keeping up with the constant changing environment. Using this basic tracking and line class structure was concluded to be ineffective at getting good results on the final video, so a brand new tracking algorithm was created from scratch using creative techniques, aimed to be robust enough to tackle the final video.

## Discussing the Advance Lane Tracking Algorithm

Using sliding windows to search for the highest amount of pixel spaces was a good start, but the biggest problem was that the left and right sides were not dependent on each other and the shapes they formed did not have to be curved splines. To address both of these problems, instead of using two seperate sliding windows, the windows were attached together by some connecting distance seperator, see the illustration below for the template. Also the the template could also slide in one direction, to the left or right, this meant it would have to produce nice looking curves and this was the results that we expected to find anyway.

# The curve template used per vertical level

  (left_window + right_window)

  (padding)                                                                (padding)           /\
   <------ |---window_width--|           (+)         |---window_width---|  -------->           |
          |---left_window---|_______________________|---right_window---|                      | level
         <------>  |----------------center_dis-----------------|   <------>                    |
       (slide_res)                                                (slide_res)                  -

To get even better results the template didnt have to just search from top to bottom in increments of 1 level, instead it could skip levels incase and interpolte incase if going level 1 by 1 it would have gone down a bad path and missed a better one, such as if there is alot of noise. Note that this assumes that lane lines both end at the top of the image, and this meant fitting the transformation so that was the case. This was the reason why the first video and third video had different outward lengths. It should also be noted that the transformed image have it so the two lane lines were nearly parallel to each other, if meeting these two conditions along with good binary images the advance lane tracker was showed to achieve good results. 

The tracker its self was not perfect, espeacially in the presence of imperfect binary images that contained missing features and extra noise, so the lines were collected and averaged over time giving nice smooth transitions. Even so it could be noticed when the tracker was getting distracted by nosie elements when it would curve towards them away from the target observed path. The curving was minimal though and its base points did very well to stay in place after much configuration inside the actual tracker. The code for the tacker was made to be very well documented so it should serve as a guide for how the algorithm is functioning on a basic level.

The end results from the advance lane tracker was producing results that were able to satisfiable for the final "harder challenge video" as well as get results better if not the same as the previous basic tracker for the other two videos. Just for fun it was thought that the car's speed could be measured by using reference template matching, measuring the vertical displacment between transformed images. The idea was completly outside the project requirments but still was a fun experiment that only sort of worked, the speed was shown to be inaccurate and flucate alot. To get this corrected more debugging would need to be done on it, and maybe just doing bare template matching with squared difference minimization is not enough to get accurate vertical sliding displacments, though it was entertaining to see the effect kind of work espeacially graphically on the overlays moving horizontal bars. In the future more work might be done on this feature, since it seems in like something should be able to give good results. 

## Final Loose Ends

At the beginning of the project work was done finding the camera matrix coeiffiecents used to undistort its camera images, the work done for that along with the camera values was saved in camera_cal, so was a rather strightforward process so not much discussion was saved for that, but inside the folder it is cool to see undistored checkerboard images, along with it matching the inside corners. All of this was important to perform accurate perspective transforms.

Also the Project called for working on images intially and outputing its binary images and fit curves, along with road offset and curvature measurments. All the results for that were saved in test_images and output images showed curves being fit along with its binary images up in the top right hand corner, along with the fitted window shape. Also in this directory were the orginal outputs that were produced with the orginal basic tracker discussed in the lecture with its same color scheme, the results were not bad. In one case the output 3 actually looked better for the basic tracker, maybe because its binary was seeing more of the right hand lane dashes. 

## Final Loose Ends

Please checkout the final results video link included at the top which showcases the performance on all 3 videos. The results on all videos was satisfying I would say but as always could use more work and polish, espeacially for creating a dynamic binary outputing, but its very easy to find more and more things to work on for this kind of open ended projects. It would also be interesting to explore more on speed tracking which could be a really cool feature if it started getting accurate results. Next step to be implemented to this project is vehicle detection, COMMING SOON.

## Porting to Python 2.7

The original author may be working on python 3.0, so there is minor errors occurring when it comes to python 2.7. This is python 2.7 compatible version.

First of all, calibrate the camera with the following, it will pickle the params in `calibration_pickle.p` to be later used.

    python camera_calibrate.py

Then for the images perform the tracking algorithm with

    python main_image_gen.py
