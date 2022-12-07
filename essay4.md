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

### test Directory
The Python tests inside the test directory mainly apply to the latest comma three hardware. The directory tests everything from exposure correction on individual frames to the processes that start the cameras. 

[test_camerad.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/test_camerad.py) contains the unit tests that set up the messaging sockets for the road camera, wide road camera, and driver camera. It ensures the daemon for the camerad package starts, stops, and logs every frame properly. Not a single frame can be skipped by any of the cameras. In addition, it checks the difference in time between each frame. If any latency is greater than 0.5 ms, the test_frame_sync unit test fails.

[test_exposure.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/test_exposure.py) covers color correction and camera snapshots. This file interacts directly with [snapshot.py](https://github.com/commaai/openpilot/blob/10085d1e3f61b472c4f25cd3e98d5ee83b40d4eb/system/camerad/snapshot/snapshot.py#L54) to get each snapshot from each camera. Each image is checked to make sure it is within the specified color ranges. If more than one image doesn’t pass, the unit test fails.

[check_skips.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/check_skips.py) doesn’t contain any actual tests. Instead, it logs which, if any, frames are skipped by any of the three cameras. It uses openpilot’s [cereal](https://github.com/commaai/cereal) messaging system to debug which sockets might not be working properly.

Lastly, [frame_test.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/frame_test.py)t validates all the imaging libraries and the camera message publishing system (cereal) is working properly.

### Chronological flow (image processing)
The Comma Device, a.k.a the camera, utilizes the camera daemon to capture both the road and driver camera. The images then undergo processes in the camerad directory such as image extraction in [snapshot](https://github.com/commaai/openpilot/tree/master/system/camerad/snapshot) and image processing in [imgproc](https://github.com/commaai/openpilot/tree/master/system/camerad/imgproc). Lastly, camera daemon uses VisionIPC and the Cereal library to send the frames data to the model daemon, which then computes predictions based on the data. The activity diagram below (source: page 148 of openPilot's official documentation) summarizes the major processes in the camerad module.

![camerad activity diagram](chronological-flowchart.png)

### imgproc
In the imgproc folder, there contains two .cl files that represent computing language files that are used to compute accelerating speed and responsiveness of applications such as the vision processing and neural network training. The main .cl file, conv.cl, contains code that processes the image. In general, the code functions by reading an RGB image and performs convolution between the kernel and the image. In image processing, a kernel is a small matrix that is used for blurring, sharpening, and edge detection in images. Convolution is a mathematical operation which involves two functions (such as f and g) and produces a third function (f \* g). However, in image processing terms, convolution refers to the process of adding each element of an image to its local neighbors, weighted by the kernel. This relates to the form of the matrix operation used in mathematical convolution.

When convolution is applied, the original image(denoted as f(x,y)) undergoes a specific operation involving the filter kernel (denoted as the omega function, w) that can produce a wide range of effects, producing a filtered image (denoted as g(x,y)). To be put in simpler terms, the conv.cl file in openPilot functions by caching local pixels of an image, comparing the image size to that of the filter size, and finally performing convolution operations. The resulting image from this process will be a weighted sum or combination of all the entries of the image matrix, with weights given by the kernel.

In utils.cc, the file basically works as a utility file that contains helper functions that help perform more complex mathematical operations. The *LapConv* function is a helper function which performs Laplace Transform Convolution, which assists in the convolution of images in the conv.cl file. Laplacian filters are used to detect edges in images. As a spacial filter, it helps to reduce spatial noise and enhance prediction in image processing. As an edge detector, laplacian filters are used to compute second derivatives on an image, measuring the rate at which first derivatives change. This determines if a change in an adjacent pixel values originate from an edge or continuous progression. This is important because each frame being fed into openPilot through the comma device has to be accurately processed continuously to constanly produce accurate and safe predictions for the vehicle. An attached image below explains how laplacian filters affect an image.

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
