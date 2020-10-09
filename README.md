# void_detection
## Problem statement
<b><i>Given an image, create an algorithm to  identify the sections where there is no product present.  </i></b>

## Solutions

This is a void detecction problem which is a part of planogram where we continuously monitor a shelf with a static camera and generate soft alerts based on item replishment and hard alerts on void detection.

There are various approches using which we can solve the problem
- Image processing
    - Void detections with a void template
    - With a reference image
- ML/AI
    - With a supervised parametric AI model


### semi supervised approach

#### with a void template

<b> Algorithm </b>
- Input -  Create a void template (an subsection of the image that denotes empty shelf)
- Read image 
- Read void template image
- scan entire image and look for areas which likely denotes an empty shelf(match void template locally in the image)
- record the bbox coordinates where there is  95% similarity between the local regions and shelf image
- Remove false detections using non max compression
- scan boxes from left to right and ssign them unique id
- Output - list of boxes depicting the top-left  and bottom-right coordinates of empty sections

#### PROS
- Simple to implement 
- Easy to interpret
- semi supervised

#### CONS
- Not very robust,To handle robustness will have to create sufficiently large no of templates which will make it slower
- Not robust to nonlinear transformations
- chances of detecting false positives are high
- Due to exact match, This will not be work properly for all environments , Can't handle scale invariance (will have to create multiscale templates to handle multiscale match),rotation,different lightning conditons, change in angle etc 
- matching thresholds might differ for different objects

#### with a reference image

<b> Algorithm </b>
- Input - Continous stream of images
- At every n time interval take a snapshot of the shelf
- Compare snapshot of time t with snapshot of time t-1(refernce image)
- Calculate similarity between the snapshots
- record the bbox coordinates where there is a difference greater than a specific threshold 
- Remove false detections using non max compression
- scan boxes from left to right and ssign them unique id
- Output - list of boxes depicting the top-left  and bottom-right coordinates of empty sections

#### PROS
- Fast 
- Accurate and fast in changes detection 

#### CONS
- Requires a continuos stream of images
- Not very robust
- Even a slight changes in the subsequent image may lead to false detections
- is not size invariant
- similarity thresold might vary for diffrent conditions

### Supervised approach

#### with image and ground truth boxes

<b> Algorithm </b>
- create training dataset with covering all possible cases of void occurance in the store
    - Record a video stream of the shelves with products and no product
    - Record positive and negative cases
    - Label a void in one frame, then use an object tracker to label the void for the rest of the frames. This would multiply the number of labels by about 30, assuming a 30 FPS camera and a void-on-screen-time of one second.
    - This would create sufficiently large training set in a very short time
    - record image and bboxes as ground truth ([image_id,[[x1,y1,x2,y2,class_id]]])
- Train a deep learning object detection model(One stage like SSD,YOLO or 2 stage like Faster RCNN, depending on speed accuracy tradeoff )
- Evaluate the model's metric 
- Deploy the model with best capacity
- Pass static image from the camera to the model
- record detections as void coordinates
- Output - list of boxes depicting the top-left  and bottom-right coordinates of empty sections

#### PROS
- Robust
- can handle scale invariances
- can handle occlusions
- can handle rotation
- Robust to nonlinear transformations
- one model can work in another similar environment

#### CONS
- Requires a large number of annotated images
- Requires to train a model
- Training time is higher
- Model deployment has to be done after training
