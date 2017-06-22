**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/vehicle_example.png
[image2]: ./examples/not_vehicle_example.png
[image3]: ./examples/hog_example.png
[image4]: ./examples/non_vehicle_hog_example.png
[image5]: ./examples/window_search.png
[image6]: ./examples/labels.png
[video1]: ./project_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

The IPython Notebook is contained in `project.ipynb`.

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the IPython notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]
![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example of each using the `grayscale` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(6, 6)` and `cells_per_block=(2, 2)`:

![alt text][image3]
![alt text][image4]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and judged the visual output for how well I was able to distinguish the vehicles in the vehicle examples. When I found that the hog output was still representing the vehicle shapes well enough, I stopped tweaking. I also attempted to distinguish how different features detected were from the vehicle images to the non-vehicle images, attempting to find a maximum difference between them.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The first step was to extract the features from the provided data set. I did this by combining the histogram of oriented gradients, spatial binning, and color histogram features into a single array for each example. This was performed for both the vehicle and non-vehicle data sets.

Then, labels were generated for all of these feature examples. The features themselves were combined and scaled, and the label sets were combined in the same order. Finally, the combined dataset was shuffled and split into training and test sets.

I then trained an SVC using sklearn and checked the accuracy of the SVC on the split test examples.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search windows from a start to stop point in both x and y directions, with a specified overlap and window size. Once I had created a set of windows to search from within an image, I could then use those windows to create feature vectors of the appropriate size to feed into my predictor and attempt classification. I also implemented a box drawing function which would take in a set of bounding boxes (expected to be around detected vehicles), and draw a blue box onto the image in those locations.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image5]

After I finished this part of the pipeline, it became clear that this was going to be far too slow to use for the video. I implemented the scaled window search, which will create a HOG version of the entire search space ahead of time, and then subsample that search space. This sped up the performance greatly, from about 3 seconds per image to 0.5 seconds per processed image.

---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_output.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video and the bounding boxes then overlaid on the last frame of video:

![alt text][image6]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I mostly ran into problems with scaling the sample windows that were being used to detect the vehicles. I found that running the sampling over two different sizes helped this some, but it took me some time to figure out how to support multiple window sizes and have the same input vector sizes. I think my pipeline is likely to fail by not detecting enough true positives, simply because I traded off speed of detecting for some accuracy. If I had more time to work, I think I would have been able to detect using more window samples from each frame.

I also did not implement a smoothing step in the video processing. This would have increased the quality of the visual output of the detected vehicles. As it is now, the video detections are quite jittery. I would have like to implement smoothing, but in the interest of time I left that for another day.

