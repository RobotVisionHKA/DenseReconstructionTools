Basalt - Visual Inertial Odometry
=================================

This is a `fork <https://github.com/RobotVisionHKA/basalt>`_ of the `Basalt <https://gitlab.com/VladyslavUsenko/basalt>`_ repository with
changes made on top so that it can be used as a VIO system for the dense MVS reconstruction network `MonoRec <https://github.com/RobotVisionHKA/MonoRec>`_.

Installation (from source)
--------------------------

.. code-block:: bash

    git clone --recursive https://github.com/RobotVisionHKA/basalt.git
    cd basalt
    ./scripts/install_deps.sh
    mkdir build
    cd build
    cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
    make -j8

Changes to the original package
-------------------------------

The primary target in order to make Basalt compatible with MonoRec was to extract
the keyframe poses (for both training and inference) and also the tracked keypoints in the keyframes
(for the sparse depth supervised loss in MonoRec, only for training).  
Here are the changes that were made on top of the existing Basalt code:  

1. A ``keyframe_data_path`` flag is added, which when valid saves the poses and keypoints in the following directory structure:  

    .. code-block:: text

        keyframe_data_path
            ├── keypoints
            |   ├── ts1.txt
            |   ├── ts2.txt
            |   └── ..
            ├── keypoints_viz
            |   ├── ts1.txt
            |   ├── ts2.txt
            |   └── ..
            └── poses
                ├── keyframe_trajectory_cam.txt
                └── keyframe_trajectory_imu.txt

    - *keypoints_viz* contains the keypoints saved from the visualization queue.  
    - *keypoints* contains the keypoints saved from the marginal data queue. (keypoints are dropped from the optimization window, hence the images in the visualization queue contain more points)  
    - *poses* contains two text files, *keyframe_trajectory_cam.txt* which contains poses in the camera frame and *keyframe_trajectory_imu.txt* which contains poses in imu frame.  

        .. note::

            - pose format is ``ts(ns) px py pz qx qy qz qw``
            - keypoints are stored in a separate text file for each timestamp. The first line contains the no. of keypoints tracked in the frame and subsequently each line contains ``x y inv_depth``.  
      
2. The ``MargDataSaver`` has been changed to take additional arguments
   ``[const string& keyframe_data_path, const basalt::Calibration<double> calib]``.
   The calibration matrix is required to convert the poses from the imu frame to the camera frame.

    .. note::
        poses in the margdata queue are in the imu frame

3. MargData Saver saves keypoints and poses for each timestamp in the MargDataQueue if the ``keyframe_data_path``
   is valid. There are 7 poses for each window and the one that matches the timestamp is saved.  

4. The ``MargDataPtr`` data structure was changed to save the keypoints as a vector.
   Whenever data is pushed into the margdata queue, keypoints are computed for the window using optical flow, 
   the ``computeProjections`` function. The computed keypoints are then added to the ``MargDataPtr``.  

5. For saving keypoints from the visualization queue, whenever data is pushed into the viz_queue, keypoints 
   are directly saved if the 'keyframe_data_path' is valid. Done in 
   the `sqrt_keypoint_vio.cpp <https://github.com/RobotVisionHKA/basalt/blob/master/src/vi_estimator/sqrt_keypoint_vio.cpp>`_ file.    

Changed files
-------------

1. `vio.cpp <https://github.com/RobotVisionHKA/basalt/blob/master/src/vio.cpp>`_
    - add flag for saving keyframe_data
    - change call to ``MargDataSaver``. add arguments - ``keyframe_data_path`` and calib data  
    - change call to initialize vio estimator. add argument - ``keyframe_data_path``  
 
2. `vio_sim.cpp <https://github.com/RobotVisionHKA/basalt/blob/master/src/vio_sim.cpp>`_  
    - change call to ``MargDataSaver``. add arguments - ``keyframe_data_path`` and calib data   

3. `marg_data_io.cpp <https://github.com/RobotVisionHKA/basalt/blob/master/src/io/marg_data_io.cpp>`_ and `marg_data_io.h <https://github.com/RobotVisionHKA/basalt/blob/master/include/basalt/io/marg_data_io.h>`_
    - save poses from the margdata queue
    - save keypoints in cam frame (if '``keyframe_data_path``' is valid)
    - save keypoints in imu frame (if '``keyframe_data_path``' is valid)  

4. `sqrt_keypoint_vio.cpp <https://github.com/RobotVisionHKA/basalt/blob/master/src/vi_estimator/sqrt_keypoint_vio.cpp>`_ and `sqrt_keypoint_vio.h <https://github.com/RobotVisionHKA/basalt/blob/master/include/basalt/vi_estimator/sqrt_keypoint_vio.h>`_
    - compute and add keypoint information when data is being pushed into the margdata queue
    - save keypoints directly before pushing into the viz_queue (if '``keyframe_data_path``' is valid)  

5. `sqrt_keypoint_vo.cpp <https://github.com/RobotVisionHKA/basalt/blob/master/src/vi_estimator/sqrt_keypoint_vo.cpp>`_, `sqrt_keypoint_vo.h <https://github.com/RobotVisionHKA/basalt/blob/master/include/basalt/vi_estimator/sqrt_keypoint_vo.h>`_ and `vio_estimator.h <https://github.com/RobotVisionHKA/basalt/blob/master/include/basalt/vi_estimator/vio_estimator.h>`_ 
    - change function definitions. add argument - ``keyframe_data_path``

6. `imu_types.h <https://github.com/RobotVisionHKA/basalt/blob/master/include/basalt/utils/imu_types.h>`_  
    - change data structure of ``MargDataPtr`` for saving keypoints  

Running VIO
-----------

Create a new folder *run* in the parent directory and run VIO from this folder to
save all the stats there, but not compulsory. However, the command below assumes that VIO 
is being executed from the *basalt/run* folder. 

.. code-block:: bash

    mkdir run
    cd run

Running VIO:  

.. code-block:: bash

    App description
    Usage: ../build/basalt_vio [OPTIONS]

    Options:
      -h,--help                   Print this help message and exit
      --show-gui BOOLEAN          Show GUI
      --cam-calib TEXT REQUIRED   Ground-truth camera calibration used for simulation.
      --dataset-path TEXT REQUIRED
                                  Path to dataset.
      --dataset-type TEXT REQUIRED
                                  Dataset type <euroc, bag>.
      --marg-data TEXT            Path to folder where marginalization data will be stored.
      --print-queue BOOLEAN       Print queue.
      --config-path TEXT          Path to config file.
      --result-path TEXT          Path to result file where the system will write RMSE ATE.
      --num-threads INT           Number of threads.
      --step-by-step BOOLEAN      Path to config file.
      --save-trajectory TEXT      Save trajectory. Supported formats <tum, euroc, kitti>
      --save-groundtruth BOOLEAN  In addition to trajectory, save also ground truth
      --use-imu BOOLEAN           Use IMU.
      --keyframe-data TEXT        Path for saving keyframe poses and keypoints.
      --use-double BOOLEAN        Use double not float.
      --max-frames UINT           Limit number of frames to process from dataset (0 means unlimited)

E.g.

.. code-block:: bash

    ../build/basalt_vio --dataset-path ../../tumvi_data/test --cam-calib ../../DenseReconstruction/basalt/data/test_1024_cropped.json --dataset-type euroc --config-path ../../DenseReconstruction/basalt/data/tumvi_512_config.json --marg-data ../../tumvi_data/test/temp_keyframe_data --show-gui 1 --keyframe-data ../../tumvi_data/test/kf_data --use-imu 1

**_the keyframe_data_path must point to the *basalt_keyframe_data* folder within the parent directory of the dataset sequence for MonoRec to be compatible i.e. able to read the poses and keypoints. That is how the dataloader is implemented.**