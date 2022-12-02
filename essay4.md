## camerad Module Overview

### test Directory
The Python tests inside the test directory mainly apply to the latest comma three hardware. The directory tests everything from exposure correction on individual frames to the processes that start the cameras. 

[test_camerad.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/test_camerad.py) contains the unit tests that set up the messaging sockets for the road camera, wide road camera, and driver camera. It ensures the daemon for the camerad package starts, stops, and logs every frame properly. Not a single frame can be skipped by any of the cameras. In addition, it checks the difference in time between each frame. If any latency is greater than 0.5 ms, the test_frame_sync unit test fails.

[test_exposure.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/test_exposure.py) covers color correction and camera snapshots. This file interacts directly with [snapshot.py](https://github.com/commaai/openpilot/blob/10085d1e3f61b472c4f25cd3e98d5ee83b40d4eb/system/camerad/snapshot/snapshot.py#L54) to get each snapshot from each camera. Each image is checked to make sure it is within the specified color ranges. If more than one image doesn’t pass, the unit test fails.

[check_skips.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/check_skips.py) doesn’t contain any actual tests. Instead, it logs which, if any, frames are skipped by any of the three cameras. It uses openpilot’s [cereal](https://github.com/commaai/cereal) messaging system to debug which sockets might not be working properly.

Lastly, [frame_test.py](https://github.com/commaai/openpilot/blob/master/system/camerad/test/frame_test.py)t validates all the imaging libraries and the camera message publishing system (cereal) is working properly.
