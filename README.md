# DenseReconstructionTools

This repository contains all the secondary scripts and tools required for dense reconstruction of an environment using [MonoRec](https://github.com/RobotVisionHKA/DenseReconstruction) and [Basalt](https://github.com/RobotVisionHKA/VisualInertialOdometry).  

The dense reconstruction network selected i.e. MonoRec requires relative camera poses for each keyframe and hence, the VIO system Basalt is chosen for the same. For supervised training of MonoRec, sparse depth maps generated from the odometry system are also required. Therefore, keypoints and poses must be extracted from Basalt and fed to the MonoRec network.  

The euroc dataset format is used for training and inference of MonoRec. The euroc dataset must be generated from a bag recording of the Realsense camera or the TUM-VI dataset. 

The main components of this repository are:  
1. Preparation of euroc dataset for MonoRec [[euroc-preparation](euroc-preparation)]
    - from the [tum-vi dataset](https://vision.in.tum.de/data/datasets/visual-inertial-dataset)
    - from bag file generated from the Realsense Depth Camera D455
    - running Basalt on the datasets for pose and keypoint extraction
2. Visualization of poses generated from Basalt (for debugging) [[trajectory_visualization](trajectory_visualization)]
3. Testing of dataloaders from MonoRec and keypoints generated from Basalt [[extras](extras)]