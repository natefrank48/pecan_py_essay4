## camerad Module Overview
### Introduction
[openpilot](https://github.com/commaai/openpilot/blob/master/README.md) is an 
open-source project focused on self-driving, with one core elements being the 
camera. The camera provides key data about the car's current position and 
the driver's awareness level, which eventually leads to the software making key decisions
on what to do next. In order to better understand how openpilot functions, we decided
to further investigate the [camera](https://github.com/commaai/openpilot/tree/master/system/camerad).

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

### test Directory
The Python tests inside the test directory mainly apply to the latest comma three hardware. The directory tests everything from exposure correction on individual frames to the processes that start the cameras. 

[test_camerad.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/test_camerad.py) contains the unit tests that set up the messaging sockets for the road camera, wide road camera, and driver camera. It ensures the daemon for the camerad package starts, stops, and logs every frame properly. Not a single frame can be skipped by any of the cameras. In addition, it checks the difference in time between each frame. If any latency is greater than 0.5 ms, the test_frame_sync unit test fails.

[test_exposure.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/test_exposure.py) covers color correction and camera snapshots. This file interacts directly with [snapshot.py](https://github.com/commaai/openpilot/blob/10085d1e3f61b472c4f25cd3e98d5ee83b40d4eb/system/camerad/snapshot/snapshot.py#L54) to get each snapshot from each camera. Each image is checked to make sure it is within the specified color ranges. If more than one image doesn’t pass, the unit test fails.

[check_skips.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/check_skips.py) doesn’t contain any actual tests. Instead, it logs which, if any, frames are skipped by any of the three cameras. It uses openpilot’s [cereal](https://github.com/commaai/cereal) messaging system to debug which sockets might not be working properly.

Lastly, [frame_test.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/frame_test.py)t validates all the imaging libraries and the camera message publishing system (cereal) is working properly.

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

### Include
* One media folder which contains 13 .h files which are all the medias that are being used. 
* Five msm folders that are sensor .h files, these all help to move camerad to the system. The main purpose of this include folder would be to make sure the sensors are being engaged as they are all being defaulted in each file. 
* Inside the media folder, there are all these headers being defined for different parts of the camera family
* All hardware stuff, like opcodes and structs being created and defined in headers to be used in the other files for sensors
* Not much "logic" coding going on in these include files, mainly all .h header files that are just, as the folder name implies, being included.
    * msm_cam_sensor.h - creates many structs and buffers for all of the cam sensors
    * msm_camsensor_sdk.h - creating the headers
    * msmb_camera.h - This one is making sure the stream isn't being over filled, creates buffers and testing situations to avoid overflows.
    * msmb_isp.h - This file includes the linux/videodev2.h header, so it goes into tracking patterns and what to do when stream stopped
    * msmb_ispif.h - Short file. Makes sure proper bits being addressed to. 

### Snapshot methods
There are many methods for snapshots to be created and processed for openpilot. The first method is extract_image. It gets the image in YUV and converts it to RGB for processing by calling yuv_to_rgb. The yuv_to_rgb method converts picture from YUV to RGB. YUV images use less bandwidth than RGB images, which will allow the image to be transported faster. The method get_snapshots recieves an image from the rear and front of the vehicle to use for processing and eventually decision making. To create an image memory and save it, jpeg_write is used. The snapshot method works like the main function for snapshots calling all of the other processes and returning the images from get_snapshots.

### Main method
The main method for snapshots works by first checking the hardware it is running on. If it is running on a PC, snapshots are not meant to be taken. Then it sets real time priority to 53. The real time priority works with interupts in the C language and is the precedence of these interupts. Next the core affinity is set to 6, and this is done on the operating system. The next thing that the main method does is check if the car is offroad. The images do not need to be captured if the car is off road because openpilot is not meant for offroading. Lastly, the method calls camera_thread in camera_commons.cc to take pictures and process them.