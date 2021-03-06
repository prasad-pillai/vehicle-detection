## Writeup

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
[sample]: ./output_images/sample.png
[hog_car]: ./output_images/hog_feature_car.png
[hog_noncar]: ./output_images/hog_feature_noncar.png
[my_sliding_win]: ./output_images/my_sliding_win.png
[sliding_win]: ./output_images/sliding_win.png
[final]: ./output_images/final.png
[video1]: ./lane_plus_car.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The dataset provided consisted of 8792 car images and 8968 non car RGB images. First i read in all the images with opencv function `cv2.imread()` and made sure to rearrange the color channels from BGR to RGB. Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![sample][sample]

Later i defined functions to extract features from the image. The function `extract_hog_features()` extracts the hog features. **The code for this step can be found in in 4th code cell of the IPython notebook**

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like. I also experimented with color and histogram features.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

** Car image hog**
![car image hog][hog_car]

** Non car image hog**
![non car image hog][hog_noncar]

#### 2. Explain how you settled on your final choice of HOG parameters.

The parameters were found experimenting on the them manually and finding the parameters which gives the most accuracy. The following features where found to give a maximum accuracy of 99.38%

Color Space: YCrCb
HOG Orientation: 8
HOG Pixels per cell: 8
HOG Cell per block: 2
HOG Channels: All
Spatial bin size: (16,16)
Histogram bins: 32
Histogram range: (0,256)
Classifier: LinearSVC
Scaler: StandardScaler

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using a feature list containing color features, histogram features and hog features of all three channels of the image in YCrCb image. Refer code cells 6-8. I extracted color, histogram and hog features and concatinated them into a single array and used this array for training.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The basic idea was to divide the image into cells of fixed(eg: 64x64) size, refer function `get_cells()` in cell 11. Then use numpy slicing to extract the sub-image inside each cell and run our trained SVM on this image slice. But the problem with this approach would be that some times (or maybe usually) the object (eg: car) we are looking for will not be entirely inside a cell. One part will be on one cell and other part will be on other and the classifier classifies both cells to be a non car image, which is not good.

![basic windows][my_sliding_win]

The approach to fix this is to jump small steps; even one pixel at a time will do. This will make our system search for literally every possible location in the image to find cars. But there is a concern that it will take a very long time for this kind of search. Thus we need a flexible way which will take not very long time but gives a good result. The function `slide_window()` (refer code cell 13),  which will give me list of windows on which i can run the classifier. The choice of overlap was made after manual trail and error considering accuracy and time to process.

![sliding windows][sliding_win]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Calculating HOG for every single window was slow, to improve the speed a hog sub-sampling, as suggested in the udacity lectures where implemented. I had to play around with the threshold value to get a valid result. The code can be found at code cell 22. The results are shown below.

![final images][final]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Here's a [link to my video result](./lane_plus_car.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  Refer code cells 19, 22 and 27.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Initially as i implemented the pipeline without sub-sampling a long time was taken to even indentify cars on a single image. I was concerned as our goal is to do this in real time ~ 30fps. But i implemented the subsampling the performance increased. 
Same thing happend when instead of adding a parameter to turn visualize on and off i kept it simply on, which resulted in generating the hog image everytime i called the hog function. This was taking 5X more than the regular time.
I also experimented with block_norm param in the hog function. I found that using L2-Hys which is going to be the default in version .15 was giving me very bad results so i have to came back to L1. 

The accuracy of the pipeline can be improved my decreasing the cell size. Here we are using a single window size only, we can do a multiple sized window search to get more reliable and robust code. This model will only work on car images so if we have other vehicles on the road then they may go undetected.

