##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/cutout5_hog_combined.jpg
[image2]: ./output_images/noncar_hog_combined.jpg
[image3]: ./output_images/frame_033.jpg
[image4]: ./output_images/frame_033_bboxes.jpg
[image5]: ./output_images/frame_033_cum_heat.jpg
[image6]: ./output_images/frame_033_heat.jpg
[image7]: ./output_images/frame_033_combined.jpg
[image8]: ./output_images/frame_033_deduped_bboxes.jpg
[image9]: ./output_images/frame_007_combined.jpg
[image10]: ./output_images/frame_020_combined.jpg
[image11]: ./output_images/frame_026_combined.jpg
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the third code cell of the IPython notebook `vehicle_tracking.ipynb` between line-numbers 28 and 31.

I started by reading in all the `vehicle` and `non-vehicle` images.  
I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:
Here is an example of one of each of the `vehicle` and `non-vehicle` classes:
![alt text][image1]
![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and settled on the above parameters as they gave a reasonabler performance on the classification of car and non-car images.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using linearSVC. The code for the same is in the 3rd cell of `vehicle_tracking.ipynb` between lines 203 and 216. I used both spatial and color histogram features.
The code for the same is in the 3rd cell between line number 16 and 26.

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search only in the most likely region where I could find the cars. I searched for smaller cars near the center and bigger cars close to the bottom of the image. I used about 5 different scales to look for cars of various sizes.  I initially implemented a version which recomputed the hog features for all the sliding windows and later used the method that was suggested in the class. The code for this is in the 3rd cell of `vehicle_tracking.ipynb` between lines 88 and 147.

![alt text][image3]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4). I have created a combined video with all the intermediate images obtained during the pipeline execution on each frame. I arranged all the intermediate stages in a grid of size (2,3)
(0,0) -> original frame
(0,1) -> with detected bounding boxes after running the classifier on sliding windows at various resolutions
(0,2) -> final thresholded heat map used for deduplicated bounding boxes for the cars
(1,0) -> heat map of the current frame
(1,1) -> cumulative heatmap of the last 5 frames
(1,2) -> final output with the deduped bounding boxes.


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are three frames and their corresponding heatmaps:

![alt text][image7]
![alt text][image10]
![alt text][image11]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. I initially had the classifier perform poorly in regions of shadow. I mitigated this and other false positive rate problem by augmenting the training data with more images in the shadowy region. This somewhat reduced the problem but not much. I then used cumulative heat maps and a threshold to remove most of the false-positives and identified contiguous blobs of thresholded region as individual cars. The pipeline currently does not identify the cars which are far but not too far. This can be improved by reducing the threshold used for the heatmaps but that would increase the false positives. 

