## camerad Module Overview

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
