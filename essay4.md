## camerad Module Overview

### cameras Directory
Inside the [cameras directory](https://github.com/commaai/openpilot/tree/master/system/camerad/cameras), there are 5 main files, each with its key elements. 
All of them have the general purpose of getting and parsing data from the camera 
and configuring the camera.

Camera common ([header file](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_common.h)) contains functions common to using the camera. 
These functions are about making the camera able to be used by other files.
* [CameraBuf](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_common.cc#L106): this contains and implements the CameraBuf class, which is a buffer 
which is required for the camera to function
* [fill_frame_data](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_common.cc#L150): this function will fill the buffer with the camera frame data
* [Threads](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_common.cc#L304): there are two threads at the end of the file: processing_thread and camera_thread. These threads are used for multithreading, which processing and input data respectively.

The main focus of Camera_qcom2 ([header file](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_qcom2.h)) is CameraState. CameraState deals with 
setting [exposure](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_qcom2.cc#L1036), 
[initialization of sensors](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_qcom2.cc#L200), 
initializing [camera](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_qcom2.cc#L589), and running the 
[camera](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_qcom2.cc#L1249). The CameraState object stores information about the current state and 
settings of the camera.

Camera_util ([header file](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_util.h)) contains low level helper functions used in the camera 
directory. Some key ones include [managing memory](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_util.cc#L123) 
and [camera control](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/camera_util.cc#L12).

[Sensor2_i2c](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/sensor2_i2c.h) contains memory locations of data on the physical camera. It 
is used to properly read and write data to the correct locations.

Finally, the [real_debayer](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/real_debayer.cl) is the only file that is not C++. It is an OpenCL file, 
and is used for image processing. Debayering is forming a proper image of all data 
received from the RGB sensors ([source](https://www.altairastro.help/why_debayer_before_stacking/)). This debayer deals with [color correction](https://github.com/commaai/openpilot/blob/master/system/camerad/cameras/real_debayer.cl#L9) 
and properly scanning in data.

### camerad Testing
 * Specifically looking at the comma3 camera unit tests which have 3 cameras.
  * Test_camerad.py - The camerad unit test sets up the messaging sockets for the 3 cameras.
    * It clears the logs then collects them by frame for each camera.
    * After clearing the beginning and end frames to account for camera startup time, it tests that each camera has logs for each frame taken. If any      frames are skipped by any camera, the test fails. 
    * After the test, the process manager shuts down the camera module.
    * In addition, it checks the difference of times between each frame. If any frame has more than a 0.5 ms delay, they are considered too laggy to be used and the test fails
  * Check_skips.py:
    * In addition, there is a file that specifically checks the messaging relation for the cameras that will print out which camera message socket has skips and how many are skipped for troubleshooting purposes.
  * Frame_test.py and Test_exposure.py
    * Frame test checks validates imaging libraries and camera message publishing system are working properly.
    * Test_exposure actually uses the snapshots from the camerad module. It then checks that the cameras are focusing properly by making the images grayscale and checking the exposure for each picture from each camera. Has built-in thresholds to check the images against. Must pass all camera checks in a row for each frame

### imgproc
* Two .cl files, representing computing language files which are used for accelerating speed and responsiveness of applications such as vision processing and neural network training.
* mainly focus on conv.cl - takes an RGB image and performs convolution between the kernel and the image
    * in image processing, a kernel is a small matrix used for blurring, sharpening, and edge detection
    * convolution is a mathematical operation which involves two functions (such as f and g) and produces a third function (f \* g)
    * However, in image processing terms, convolution refers to the process of adding each element of an image to its local neighbors, weighted by the kernel. This relates to the form of the matrix operation used in mathematical convolution
    * When convolution is applied, the original image(denoted as f(x,y)) undergoes a specific operation involving the filter kernel(denoted as the omega function, w) that can produce a wide range of effects, producing a filtered image(denoted as g(x,y))
    * Therefore, the conv.cl file in openPilot functions by caching local pixels of an image, comparing the size to that of the filter size, and finally performing convolution operations
    * This results in a resulting image which would be a weighted sum or combination of all the entries of the image matrix, with weights given by the kernel
* pool.cl - calculates the variance in each subregion of an image
* utils.cc - contains utility helper functions that help perform more complex mathematical operation
    * LapConv which is assumed to be a helper function to perform Laplace Transform Convolution
    * Laplacian filters are used to detect edges in images. As a spacial filter, it helps to reduce spatial noise and enhance prediction in image processing
    * As an edge detector, laplacian filters are used to compute second derivatives on an image, measuring the rate at which first derivatives change.
     This determines if a change in adjacent pixel values originate from an edge or continuous progression.
     ![Laplace image](laplace-image.png)

### Snapshot
* methods
  * extract_image() - get the image in YUV and converts it to RGB for processing by calling yuv_to_rgb()
  * get_snapshots() - gets an image from the rear of the vehicle and the front
  * jpeg_write() - creats an image memory and saves it
  * snapshot() - works like the main function for snapshots calling all of the other processes and returning the images from get_snapshots()
  * yuv_to_rgb() - converts picture from YUV to RGB

### init
* nothing in here?

### main
* checks hardware
* sets real time priority
* sets core affinity - this is done on the operating system
* does not work offroad
* calls camera_thread in camera_commons.cc to take pictures and process them