# OpenCV with Raspberry Pi

"What is OpenCV?", you may be asking. OpenCV is a computer vision library, CV, that is open source, Open. Originally created in C++ it has now been ported to hundreds of languages, including Python, Java, C#, Swift, and many more. It is exteremely robust to use which is why it is the perfered vision library for many. Companies have made their own CV vision libraries that are now used as a replacement to OpenCV, like Vision for Swift, but for many hobbyists and researchers, OpenCV is the way to go.

# Setting up OpenCV

We will be using OpenCV with Python today so the installation is fairly simple. We will be using a prebuilt version of OpenCV because we are limited in time, but it is recommended you build it yourself. This would take between 6-10 hours.

To install OpenCV we need to open a terminal window.

Once in the terminal we want to make sure everything is up to date by typing `sudo apt-get update`. This command updates tools and programs that are on the Pi, it makes sure everything will run smoothly.

When that finishes we can install OpenCV with `sudo apt-get install python-opencv`. This installs the Python 2 version of OpenCV, cv2.

This next step only needs to be done if you are using the PiCamera. The PiCamera is not enabled by default so we have to tell Raspbian to allow it. To do this we need to edit the Raspberry Pis config.

To enter the Pi Config you will type `sudo raspi-config` in the terminal window. This opens a menu where you can edit the Pis settings.

We want to go to option 5, Interfacing Options. You use the arrow keys and enter to navigate.

We then want to select Pi Camera, select Yes, and then Ok. This will enable the Pi Camera.

To leave the config tap the right arrow twice to navigate to finish. Select it and then it will prompt you to reboot your Pi. When thats finished you will be ready to go!

OpenCV has many ways to detect shapes and items. You could filter the images to just be contours, and then see what contours look like what you are looking for, you could only allow one color if you were looking for something with a unique color. There are many ways to processing images in OpenCV. We will be using a Cascaded Classifer. A Classifer is a prebuilt setup of instructions for OpenCV to look for, this could include color filtering and contour detection. Building a classifer requires a fairly powerfull computer, several hours to days, and thousands of sample photos. We will be using a face classifier which was built by giving OpenCV thousands of pictures of peoples faces and using a Machine Learning technique it figured out what makes them similer. OpenCV can then use this to look at any new photos and tell you if there is a face in them and where that face is. Our Classifier is called FrontalFace, because we want to detect the front of someones face.

To download this Classifier we will open a terminal window and navigate to the Desktop with `cd Desktop`

Once we are there we can download the classifier with `curl -O https://goo.gl/vG3PFo`. This link has been shortned with a URL shortner but this command is going to the OpenCV github page and downloading the definition.

You can open the classifier to see how it works, defining points on your face that are easily recognizable along with rectangles caused by shadows in our face. Make sure not to edit anything in this file or it will not work. If it asks you to save when closing make sure to hit no.

# Programming Our Script

To create our script click on the Raspberry in the top left corner -> Programming -> Python 2 (IDLE). This will open IDLE which is the default IDE for Python.

Then we need to do File -> New File and we will have a blank python script.

Because some people may have PiCameras and some may have USB Cameras the instructions will be a little different for each. The core OpenCV will be the same but how you access the camera will change.

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

We now need to setup our cameras.

Step one is to create our cameras in the script.

With a PiCamera you use: `cap = PiCamera()` 

With a USB Webcam you use: `cap = cv2.VideoCapture(0)`

What these do is allow use to reference our camera when trying to access its feed.

The Raspberry Pi is not the most powerfull device out there. Because of this we need to lower the resolution of our camera.


With a PiCamera you use: `cap.resolution = (320, 240)`

With a USB Webcam you use:
```python
cap.set(3, 320)
cap.set(4, 240)
```

There isn't a direct way to set a USB Webcams resolution so by setting the 3rd and 4th value we can modify the resolution of the incoming frame. Every image has some meta data with the resolution, name, location. The 3rd and 4th position are the width and height of a USB Webcam.

These lines set our cameras resoltuion to 320x240, small but still big enough to see whats going on.

For the Pi Camera to work you also need a few more items:
```python
cap.framerate = 30
rawCapture = PiRGBArray(cap, size=(320, 240))
```

This sets the PiCamera to have a framerate of 30, and assigns where to save the raw image for each frame.

If you have a PiCamera your setup should look like this:
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
If you have a USB Webcam your setup should look like this:
```python
import cv2
import time

cap = cv2.VideoCapture(0)
cap.set(3, 320)
cap.set(4, 240)
```

We should save our script now so we don't lose it on accident later. Go to the top and file -> save to the desktop of the Pi. We want it be here because thats where we saved our classifier to.

Once saved we can make our reference to the classifer: `classifer = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')`

This creates a new OpenCV Classifier using the file we downloaded earlier.

And then we can tell the program to sleep a small amount of time to get the cameras up and running with `time.sleep(1)`

Now we have all our inital variables created and ready to go. We can now jump into processing our video!

To get access to our camera we have to loop through all the frames our camera is capturing.

With a PiCamera you use: `for frame in cap.capture_continuous(rawCapture, format='bgr', use_video_port=True):`

With a USB Webcam you use: `while True:`

The reasons the PiCamera has so much additional script is because the PiCamera is not a video camera. So we have to capture a photo every frame and process it seperatly rather than processing a stream like we do with the USB Webcam.

Inside the loop we need to access this frames image for each camera.

With a PiCamera you use: `img = frame.array`

With a USB Webcam you use: `ret, img = cap.read()`

We can then show this image to the user using `cv2.imshow('Image', img)`

On the Pi Camera we have to clear out the raw data so we have to add `rawCapture.truncate(0)`

If you have a PiCamera your loop should look like this:
```python
for frame in cap.capture_continuous(rawCapture, format='bgr', use_video_port=True):
    img = frame.array
    cv2.imshow('Image', img)
    rawCapture.truncate(0)
```
If you have a USB Webcam your loop should look like this:
```python
while(True):
    ret, img = cap.read()
    cv2.imshow('Image', img)
```

To run our program we want to go to the terminal and navigate to the desktop with `cd Desktop`

Then run `python nameofprogram.py` and your program should start and you should see your camera feed. To exit hit Ctrl + C in the terminal window

To add face detection we need to filer our image down to gray scale. We can do this right under where we access the frames image. 

To convert to grayscale we can use `gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)`

This takes our img and converts it from the standard format, BGR, to a Gray Format

To detect faces we will use the detectMultiScale function, with `faces = classifer.detectMultiScale(gray, 1.3, 5)`. The 1.3 and the 5 are the min and max scale we should look for faces at. This helps filter out small shadows or reflected light that it might think is a face, this doesn't always work.

Our script is now looking for faces each frame, but we can't tell its doing anything. A basic way to see if we see a face would be to print out how many faces we see, but OpenCV supports drawing shapes on images before they are shown. We will use this to draw a square around any face we see.

To draw the square we use:
```python
for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
```
This chuck of code finds the X and Y coordinate of the face on the screen as well as the width and height of each face. We then draw a square by plotting two points opposite of each other. The first point is easy, the X and Y position. The second point we just add the width to the X and the Height to the Y to get our face. We also set the color of the square with R, G, and B. Right now its Red, you can put whatever color you want here.

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
If q is pressed when close the capture, with close or release depending on the camera you are using. And then we close all windows to quit our program, the break exits the loop and stops our script.

You should now be able to run the program and see a square around any faces it sees. The Raspberry Pi is limited in power so this will be slow, but it works!
