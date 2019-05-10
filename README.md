<div align="center">
  <h1>Real-time Finger Detection</h1>
</div>

<div align="center">
  <strong>Written entirely in Python using OpenCV</strong>
</div>  

<div align="center">
  <img src="https://media.giphy.com/media/XeMmEZSzLK0mLwzhPG/giphy.gif" alt="Real-time finger detection example">
</div>
  
<div align="center">  
  <sub>Made for CS390 - Machine Perception</sub>
</div>

## Table of Contents
- [Background](#background)
- [Requirements](#requirements)
- [Features](#features)
- [Python Code Samples](#python-code-samples)
- [Future Improvements](#future-improvements)

## Background
The purpose of this project was to create a real-time finger detection program using Python and OpenCV. To accomplish this, we made use of **convex hulls** - a convex closure of a set *X* of points in a Euclidean space that is the smallest convex set that contains *X*. 

The image below shows a high-level example of how convex hulls work. For our project, the convex hull was used to segment the hand and find the fingers by looking into extremities which touches the edge of the convex hull.
<div align="center">
  <img src="https://miro.medium.com/max/1354/1*F4IUmOJbbLMJiTgHxpoc7Q.png" alt="Convex hull example">
</div>
  
## Requirements
1. [Anaconda 2019.03](https://www.anaconda.com/distribution/) (Python 3.7 version)
2. [OpenCV](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_tutorials.html)  
	```conda install -c conda-forge opencv```
3. [imutils](https://github.com/jrosebr1/imutils)   
	```conda install -c pjamesjoyce imutils```
## Features
- **Finger derection** in real-time by using a webcam
- **Visually displays** various interesting aspects (palm center, extremities, etc.)
- In-depth use of **OpenCV**
- **Region of interest** (ROI) detection
- **Finger-frame summation** to account for errors
- **Skin detection** by using a weighted background algorithm

## Python Code Samples
All code for this project resides in a single Jupyter notebook.  

There are four functions:
### Finding the background
```python
def findBackgroundMatrix(img, weight):
    global background_start
    
    # Initialized the background
    if background_start is None:
        background_start = img.copy().astype("float")
        return

    # Adds the image to the accumulator
    cv2.accumulateWeighted(img, background_start, weight)
    cv2.imshow("Background", img)
```

### Finding the hand
```python
def findHand(img, threshold=25):
    global background_start
    
    # Gets the difference between the background and frame
    difference = cv2.absdiff(background_start.astype("uint8"), img)

    # Thresholds the difference to get the foreground (hand)
    threshold_img = cv2.threshold(difference, threshold, 255, cv2.THRESH_BINARY)[1]
    global threshold_copy
    threshold_copy = copy.deepcopy(threshold_img)
    

    # Gets the contours in the thresholded image
    # RETR_EXTERNAL only returns extreme outer contour flags
    # CHAIN_APPROX_SIMPLE removes redundant points and compresses the countour (saves memory too)
    contours = cv2.findContours(threshold_img.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    contours = imutils.grab_contours(contours)

    # If no countours
    if len(contours) == 0:
        return
    else:
        # The maximum countour will be the hand
        segmentedHand = max(contours, key=cv2.contourArea)
        return (threshold_img, segmentedHand)
```

### Getting the number of fingers
```python
def getFingers(threshold_img, segmentedHand):
    
    # Gets the Convex Hull of the segmented hand
    convex_hull = cv2.convexHull(segmentedHand)

    # Gets extreme points from the Convex Hull
    convex_hull_top    = tuple(convex_hull[convex_hull[:, :, 1].argmin()][0])
    convex_hull_bottom = tuple(convex_hull[convex_hull[:, :, 1].argmax()][0])
    convex_hull_left   = tuple(convex_hull[convex_hull[:, :, 0].argmin()][0])
    convex_hull_right  = tuple(convex_hull[convex_hull[:, :, 0].argmax()][0])

    # Gets the center of the palm
    palmX = (convex_hull_left[0] + convex_hull_right[0]) // 2
    palmY = (convex_hull_top[1] + convex_hull_bottom[1]) // 2
    
    # Draws the center of the palm & region
    cv2.circle(frame_copy, (palmX+350, palmY+10), 5, (0,0,255), -1)
    cv2.circle(roi_copy, (palmX, palmY), 5, (0,0,255), -1)
    
    # Gets the euclidean distances between palm center and extreme points
    eucl_distance = pairwise.euclidean_distances([(palmX, palmY)], Y=[convex_hull_left, convex_hull_right, convex_hull_top, convex_hull_bottom])[0]
    
    # Gets the maximum euclidean distance
    max_eucl_distance = eucl_distance[eucl_distance.argmax()]

    # Calculates the radius with the euclidean distance
    eu_rad = int(0.7 * max_eucl_distance)
    
    # Gets the circumference of the palm circle
    cir = (2 * np.pi * eu_rad)

    # Gets the hand ROI
    hand_roi = np.zeros(threshold_img.shape[:2], dtype="uint8")
    
    # Draws the hand ROI
    cv2.circle(hand_roi, (palmX, palmY), eu_rad, 255, 1)
    cv2.circle(frame_copy, (palmX+350, palmY+10), eu_rad, 255, 1)
    cv2.circle(roi_copy, (palmX, palmY), eu_rad, 255, 1)
    
    # Gets the cuts obtained through a bit-wise AND between the hand (thresh) and hand ROI
    hand_roi = cv2.bitwise_and(threshold_img, threshold_img, mask=hand_roi)

    # Gets the contours in the hand ROI
    # CHAIN_APPROX_NONE stores all the boundary points, unlike CHAIN_APPROX_SIMPLE
    contours = cv2.findContours(hand_roi.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_NONE)
    contours = imutils.grab_contours(contours)
    
    # Used for accumulating fingers
    num_fingers = 0

    # For each contour found
    for contour in contours:
        
        # Gets the bounding box of the contour
        (x, y, w, h) = cv2.boundingRect(contour)

        # Considered a finger if it's not the wrist
        if ((palmY + (palmY * 0.25)) > (y + h)) and ((cir * 0.25) > contour.shape[0]):
            num_fingers += 1
            
            # Draws the finger points
            cv2.circle(frame_copy, (x+350, y+10), 4, (255,255,255), -1)
            cv2.circle(roi_copy, (x, y), 4, (255,255,255), -1)
            cv2.line(roi_copy,(palmX,palmY),(x,y),(156,127,254),1)

    return num_fingers
```

### Main function
```python
if __name__ == "__main__":
    
    # Gets the webcam as the capture source
    camera = cv2.VideoCapture(1)
    
    # Variables for use
    camera_frames = 0
    finger_frames = 0
    finger_avgerage = 0
    finger_avgerage_final = 0
    accumulated_weight = 0.5
    
    while(True):
        
        # Gets the current frame
        (grabbed, frame) = camera.read()
        
        # Resizes the frame
        frame = imutils.resize(frame, width=700)
        
        # Flips the frame
        frame = cv2.flip(frame, 1)
        
        # Copys the frame
        global frame_copy
        frame_copy = frame.copy()

        # Gets the height and width of frame
        (height, width) = frame.shape[:2]
        
        # ROI points
        # 10 = top
        # 225 = bottom
        # 350 = right
        # 590 = left
        roi = frame[10:225, 350:590]
        global roi_copy
        roi_copy = roi.copy()
        
        # Converts the ROI to grayscale
        roi_grayscale = cv2.cvtColor(roi, cv2.COLOR_BGR2GRAY)
        
        # Blurs the converted ROI
        roi_grayscale = cv2.GaussianBlur(roi_grayscale, (7, 7), 0)
        
        # Calibrates the background (weighted average model) by looking until a threshold is reached
        if camera_frames <= 30:
            findBackgroundMatrix(roi_grayscale, accumulated_weight)
                
        else:
            
            # Gets the segmented hand region
            hand_region = findHand(roi_grayscale)
            
            # If the hand is already segmented
            if hand_region is not None:

                # Unpack the segmented hand into its two parts
                (threshold_img, segmentedHand) = hand_region
                
                # Draw the segmented hand and display the ROI frame
                cv2.drawContours(frame_copy, [segmentedHand + (350, 10)], -1, (0, 255, 255))
                
                # If the number of frames in which a hand is in the ROI is less than 250
                if finger_frames <= 250:
                    
                    # Accumulate the number of fingers per frame
                    finger_avgerage += getFingers(threshold_img, segmentedHand)
                else:
                    
                    # Sum the number of fingers per frame over the period
                    finger_avgerage_final = finger_avgerage / 250
                    
                    # Resets the averages
                    finger_frames = 0
                    finger_avgerage = 0
                
                # Goes to the next frame
                finger_frames += 1
                
                # Gets the number of fingers per frame
                fingers = getFingers(threshold_img, segmentedHand)
                
                # Draws onto the thresholded image (mask)
                cv2.putText(threshold_copy, str(fingers), (10,40), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,255), 2, cv2.LINE_AA)
                cv2.putText(threshold_copy, str(finger_avgerage_final), (210,40), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,255), 2, cv2.LINE_AA)
                cv2.putText(threshold_copy, str(finger_frames), (10,200), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255,255,255), 1, cv2.LINE_AA)
                threshold_copy = imutils.resize(threshold_copy, width=700)
                cv2.imshow("Masked", threshold_copy)
                
                # Draws onto the ROI image (segmented hand in RGB)
                cv2.putText(roi_copy, str(fingers), (10,40), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,255), 2, cv2.LINE_AA)
                cv2.putText(roi_copy, str(finger_avgerage_final), (210,40), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,255), 2, cv2.LINE_AA)
                cv2.putText(roi_copy, str(finger_frames), (10,200), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255,255,255), 1, cv2.LINE_AA)
                roi_copy = imutils.resize(roi_copy, width=700)
                cv2.imshow("Hand", roi_copy)
                
        # Draws the segmented hand onto the frame
        cv2.rectangle(frame_copy, (590, 10), (350, 225), (255,255,255), 2)
        
        # Displays the segmented hand
        cv2.imshow("Input", frame_copy)
        
        # Goes to the next frame
        camera_frames += 1
        
        # Waits for the user to exit
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
# Releases the capture
camera.release()
cv2.destroyAllWindows()
```

## Future Improvements
- [ ] Add compatability with all operating systems
- [ ] Add a GUI with C++

## License
[MIT](https://tldrlegal.com/license/mit-license)
    

