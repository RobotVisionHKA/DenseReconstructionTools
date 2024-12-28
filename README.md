# Monocular Dense 3D Reconstruction Toolkit

**Documentation and tutorials are available [here](https://hardik01shah.github.io/DenseReconstructionTools/#).**

## Introduction

Dense 3D reconstruction using a single camera is a critical step in scene understanding and visualizing real-world environments. The process involves generating dense depth maps that capture detailed 3D representations of a scene.

A dense reconstruction pipeline uses monocular images, respective image poses, and sparse depth maps (for supervision), and outputs high-resolution depth maps and reconstructed 3D environments. The technique relies heavily on accurate camera motion estimation and sparse 3D geometry, making Visual-Inertial Odometry (VIO) a crucial component.

Visual-Inertial Odometry (VIO) combines camera data and inertial measurements to estimate the trajectory of a moving camera. It provides Relative Camera Poses required for frame alignment in dense reconstruction and Sparse Depth Maps i.e. the triangulated 3D points used in pose estimation using 2D-3D point correspondences.

## MonoRec and Basalt: The Reconstruction Pipeline

This toolkit integrates two advanced tools, [MonoRec](https://github.com/Brummi/MonoRec) and [Basalt](https://gitlab.com/VladyslavUsenko/basalt), to create a streamlined pipeline for dense reconstruction. We introduce changes to the original tools to enable seamless integration and work with forks of the original repositories:
 - [MonoRec Fork](https://github.com/RobotVisionHKA/MonoRec): A UNet-style network for dense depth map generation from posed monocular images.
 - [Basalt Fork](https://github.com/RobotVisionHKA/basalt): A Visual-Inertial Odometry (VIO) and mapping system that provides accurate camera poses and sparse depth maps from a sequence of monocular images (and optionally IMU data).

### Integration:

Basalt is used to extract keypoints, camera poses, and sparse 3D geometry, which are fed into MonoRec to produce dense depth maps and 3D Reconstructions(visualized as pointclouds with color information) of the scene. The dense depth maps along with the camera poses are used to generate the 3D reconstructions. Together, they form an end-to-end pipeline for dense 3D reconstruction using monocular images.

## Project Capabilities

This repository includes tools and scripts for:
1. Preparation of the EuRoC Dataset for MonoRec [[euroc-preparation](./euroc-preparation/)]:
    - Conversion of the TUM-VI dataset into EuRoC format.
    - Generation of EuRoC-format data from bag files recorded with the Intel RealSense Depth Camera D455.
    - Running Basalt to extract camera poses and keypoints for training and inference.
2. Pose Visualization [[trajectory_visualization](./trajectory_visualization/)]:
    - Debugging and visualization of camera trajectories generated by Basalt.
3. Testing and Validation [[extras](./extras/)]:
    - Validation of MonoRec’s dataloaders.
    - Verification of keypoints extracted by Basalt.

We hope this toolkit serves as a valuable resource for researchers and developers working on monocular dense reconstruction projects.

## Quick Start

### Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/RobotVisionHKA/DenseReconstructionTools.git
    cd DenseReconstructionTools
    ```
2. Clone the submodules:
    ```bash
    git submodule update --init --recursive
    ```

3. Install the required dependencies:
    ```bash
    conda env create -f environment.yml
    conda activate dense_reconstruction_toolkit
    ```
4. Prepare the data (Realsense Bag / Tum-vi) in Euroc Format. Refer to the respective READMEs for instructions:
    - [Realsense Bag](./euroc-preparation/realsense_bag/README.md)
    - [TUM-VI](./euroc-preparation/tumvi/tumvi_rectification/README.md)

### Running the Pipeline

1. Run Basalt to extract camera poses and keypoints:
    ```bash
    cd euroc-preparation/
    python3 run_basalt_euroc.py <data_path> ./basalt/
    ```
2. Run MonoRec to generate dense depth maps aggregated to a pointcloud. Make sure to update the data paths in the configuration file:
    ```bash
    cd MonoRec/
    python3 create_pointcloud.py --config configs/test/pointcloud_monorec_tumvi.json
    ```

### Visualization

1. Visualize the camera trajectory from Basalt:
    ```bash
    cd trajectory_visualization/
    python3 visualize_trajectory.py <data_path>
    ```

2. Visualize the generated pointcloud using [CloudCompare](https://www.danielgm.net/cc/) or [Open3D](http://www.open3d.org/):
    ```bash
    cloudcompare <pointcloud_path>
    ```
    or
    ```bash
    python3 MonoRec/rgbd2pcl.py --config MonoRec/configs/test/pointcloud_monorec_euroc.json
    ```

## Contributors

This project is mainly developed and maintained by [Hardik Shah](https://github.com/hardik01shah) and [Niclas Zeller](https://github.com/NiclasZeller). Issues and contributions are very welcome at any time.
