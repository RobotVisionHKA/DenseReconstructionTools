Realsense Recording and Conversion to EuRoC
===========================================

A sequence is recorded from the RealSense Depth Camera D455.
This is done using the *realsense-viewer* tool from `librealsense <https://github.com/IntelRealSense/librealsense>`_.

LibRealsense Installation
-------------------------

Installation of *librealsense* can be followed from `here <https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md>`_
which supports Ubuntu LTS kernels 4.4, 4.8, 4.10, 4.13, 4.15, 4.18*, 5.0*, 5.3* and 5.4. Installation from source
can also be done from `here <https://github.com/IntelRealSense/librealsense/blob/master/doc/installation.md>`_.
A source-code build of the SDK supports newer kernels (5.8 and 5.11). Unsupported kernels can be used with the SDK
but there may be unpredictable consequences in regards to stability. Finally, the SDK was installed
(partially, some dependency matches failed and errors due to unsupported kernel version) on
the kernel version 5.17.5. `[ref] <https://github.com/IntelRealSense/librealsense/issues/10581>`_

Data Collection
---------------

In total, 3 sequences were recorded whose bag file can be found `here <https://drive.google.com/drive/folders/16aTFunFtIfq2zIJ8XRTeHbgc-WwfgCHx?usp=sharing>`_:

1. **hka-indoors-1**: Sequence recorded indoors outside the lab on the left

2. **hka-indoors-2**: Sequence recorded indoors outside the lab on the right

3. **hka-outdoors-1**: Sequence recorded in front of Building F, around the hole  

During recording, the following sensors were streaming data:  

1. StereoModule:  
    - Left and Right stereo images from Infrared cameras
    - Projector - :code:`off`
    - Depth feed - :code:`off`
  
2. RGB Camera:
    - In the future, RGB frames can be used for the MonRec dense reconstruction network instead of the left stereo image
  
3. Motion Module:
    - Gyroscope
    - Accelerometer  

**Important parameters to take note of while recording:**  

1. In the indoor sequences there was a lot of motion blur due to low exposure times and as a result, basalt vo was failing to track keypoints. A solution would be to reduce the Auto Exposure limit setting to 5ms (or even lower) in the StereoModule settings.  

2. The Accel and Gyro readings are not synchronized. [`ref <https://github.com/IntelRealSense/librealsense/issues/3921>`_]

3. The RGB and stereo images are not synchronized. However, they can be synced by turning the AE Priority off in the RGB Module. [`ref <https://github.com/IntelRealSense/librealsense/issues/774#issuecomment-454358565>`_]

Conversion of Bag file to Euroc
-------------------------------

The recording from the *realsense-viewer* is by default saved as a bag file.
Sensor information must be extracted from the bag file into the proper Euroc format so that Basalt (VIO) and
MonoRec (dense reconstruction) can be run on them. This is done using the python
package `rosbag <https://pypi.org/project/rosbags/>`_ in the `bag2euroc.py <https://github.com/hardik01shah/DenseReconstructionTools/blob/master/euroc-preparation/realsense_bag/bag2euroc.py>`_ script.  
**Basalt requires that the Accel and Gyro values are synced and that both sensor information should be available for a timestamp at the same frame rate.**
Hence, linear interpolation is done of the Accel values of the two nearest timestamps for each Gyro value.  
Running the script:

.. code-block:: text

    usage: bag2euroc.py [-h] bag_path euroc_path

    Python code for conversion of realsense bag file to euroc format.

    positional arguments:
      bag_path    path to rosbag file
      euroc_path  path to euroc dataset

    optional arguments:
      -h, --help  show this help message and exit

    E.g.
    python3 bag2euroc.py ./final.bag ./euroc/

There were other methods of extracting the data from a bagfile:  
1. `bag2euroc_playback.py <https://github.com/hardik01shah/DenseReconstructionTools/blob/master/euroc-preparation/realsense_bag/bag2euroc_playback.py>`_ is a rospy script (requires a ros installation).
The bag file needs to be played using *rosbag play* and the script is basically a ROS subscriber which subscribes to the topics of interest and saves sensor information.  

    .. code-block:: text

        usage: (execute in 3 different terminals)
        1. roscore
        2. rosbag play -d 10 test.bag
        3. python3 bag2euroc.py <path to euroc dataset>

        * the second command waits for 10s after advertising the topics before streaming the data on the topics. It is for the subscriber to have enough time to latch onto the topics.

2. Another python package *bagpy* is used to extract the data into .csv files.
Demonstrated in `bagpy_test.ipynb <https://github.com/hardik01shah/DenseReconstructionTools/blob/master/euroc-preparation/realsense_bag/bagpy_test.ipynb>`_.