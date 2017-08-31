Starting with a fresh install of raspbian

We first need to install opencv, you can do this by typing `sudo apt-get install python-opencv` in a terminal window

When that completes opencv will be successfully installed

If you are using a PiCamera we need to enable it in the Raspberry Pi Config. To do this open a terminal window and type `sudo raspi-config`. Inside the window you need to navigate to `5 Interfacing Options` using the arrow keys and enter to select then `P1 Camera` and hit `<Yes>`. You can then hit `<Ok>`. Click the right arrow twice to navigate to `<Finish>` and it will prompt you to restart your Pi.

In order to detect faces we also need to know what a face looks like. This is done with classifiers. Classifiers are xml documents that contain definitions of things. We will be using one called frontal face because we want to detect when someones face is in the view.

To download this Classifier we will open a terminal window and navigate to the Desktop with `cd Desktop`

Once we are there we can download the classifier with `curl -O https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml`

If you open the file you can see how the file works. There are a bunch of nodes and rectangles that defines the face. On a human we have rectangles all over our face that a computer can be trained to spot. These rectangles come from the angles of our skin that reflect light and the angles that absorb light, the bright parts and the shadowy parts.

Now that we have our file and opencv installed we can start programming our script. The script will branch in certain places depending on if you are using a USB Webcam or a Picamera. I will explain what each step is doing and then what to do for each device or if you should do it for all devices.

We now need to make a new python script. To do this we can click on the Raspberry in the top left corner -> Programming -> Python 2 (IDLE). This will open IDLE which is the default IDE for Python.

Then we need to do File -> New File and we will have a blank python script.

We need to import some libraries to be able to use opencv and the cameras. To start, at the top of the file we can type:
```python
import cv2
import time
```
This will import cv2, or our OpenCV library that we will use to filter and then detect faces in an image.

It will also import time, a library used to work with time. We will use it to allow the camera to have time to setup before we start using it.

If you have a PiCamera we also need to import two additional libraries:
```python
from picamera.array import PiRGBArray
from picamera import PiCamera
```
These libraries will allow us to read the data coming back from the pi camera

We now need to setup our cameras. Each camera type has a different setup so I will walk through both at the same time.

Step one is to create our camera with a PiCamera you use: `cap = PiCamera()` and with a USB Webcam you use `cap = cv2.VideoCapture(0)`. Each of these lines do the same thing, enable a stream from our camera.

Our next step is to set the resoultion of the camera. This is done in two seperate ways for each camera. On the Pi Camera you use `cap.resolution = (320, 240)`.

For the USB Camera you have to use:
```python
cap.set(3, 320)
cap.set(4, 240)
```
There isn't a direct way to set a USB Webcams resolution so by setting the 3rd and 4th value we can modify the resolution of the incoming frame. Every image has some meta data with the resolution, name, location. The 3rd and 4th position are the width and height of a USB Webcam.

Each of these sets the camera resolution to 320 x 240. We have to use a relativly small resolution or the Pi wont be able to process the images.

The PiCamera also requires two additonal components, framerate, and a raw file. This is done with:
```python
cap.framerate = 30
rawCapture = PiRGBArray(cap, size=(320, 240))
```

This completes our camera setup, if you have a PiCamera it should look like this:
```python
import cv2
import time
from picamera.array import PiRGBArray
from picamera import PiCamera

cap = PiCamera()
cap.resolution = (320, 240)
cap.framerate = 30
rawCapture = PiRGBArray(cap, size=(320, 240))
```
Or a USB Webcam:
```python
import cv2
import time

cap = cv2.VideoCapture(0)
cap.set(3, 320)
cap.set(4, 240)
```

We can now define our classifer that we will be using to detect our face. First we should save this script to the desktop with File -> Save and save it to the desktop with a name you will remember

Now we can create our detector, this will be the same with either camera: `detector = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')`

This creates a new OpenCV Classifier using the file we downloaded earlier.

And then we can tell the program to sleep a small amount of time to get the cameras up and running with `time.sleep(1)`

Now we have all our inital variables created and ready to go. We can now jump into processing our video!

To get access to our camera we have to loop through all the frames our camera is capturing. With the USB Webcam and OpenCVs Camera tool it is a continous feed so we can just use a `while(True):` loop.

For PiCamera you can only capture frames one at a time, so to get a video feed we have to do a little more work. `for frame in cap.capture_continuous(rawCapture, format='bgr', use_video_port=True):`

Inside these loops we need to access the image the webcam is outputting. This done on the PiCamera with `img = frame.array` and on the USB Webcam with `ret, img = cap.read()`

We can then show this image to the user using `cv2.imshow('Image', img)`

On the Pi Camera we have to clear out the raw data so we have to add `rawCapture.truncate(0)`

To run our program we want to go to the terminal and navigate to the desktop with `cd Desktop`

Then run `python nameofprogram.py` and your program should start and you should see your camera feed. To exit hit Ctrl + C in the terminal window

Here is what your loop should loop like with PiCamera:
```python
for frame in cap.capture_continuous(rawCapture, format='bgr', use_video_port=True):
    img = frame.array
    cv2.imshow('Image', img)
    rawCapture.truncate(0)
```
Or on USB Webcam:
```python
while(True):
    ret, img = cap.read()
    cv2.imshow('Image', img)
```

To add face detection we need to filer our image down to gray scale. We can do this right under where we access the webcam. 

To convert to grayscale we can use `gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)`
This takes our img and converts it from the standard format, BGR, to a Gray Format

To detect faces we will use the detectMultiScale function, with `faces = detector.detectMultiScale(gray, 1.3, 5)`

We now detect any faces that are in the frame.

To make it clear when we see a face we can draw a square around them with:
```python
for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
```
This will draw a rectangle where the pi sees a face.

One more thing we are going to add is a way to easily exit our program by adding a close key
This can be done by checking if the q key is pressed:
```python
if cv2.waitKey(1) & 0xFF == ord('q'):
    # For PiCamera
    cap.close()
    # For USB Webcam
    cap.release()
    cv2.destroyAllWindows()
    break
```

You should now be able to run the program and see a square around any faces it sees. The Raspberry Pi is limited in power so this will be slow, but it works!

