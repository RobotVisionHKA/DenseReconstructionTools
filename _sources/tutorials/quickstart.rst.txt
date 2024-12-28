Quick Start
===========

Data Preparation
----------------


Prepare the data (Realsense Bag / Tum-vi) in Euroc Format. Refer to the respective documentation for instructions:

    - `Realsense Bag <../documentation/euroc-preparation/realsense_bag>`_
    - `TUM-VI <../documentation/euroc-preparation/tumvi>`_


Running the Pipeline
--------------------

1. Run Basalt to extract camera poses and keypoints:

    .. code-block:: sh

        cd euroc-preparation/
        python3 run_basalt_euroc.py <data_path> ./basalt/

2. Run MonoRec to generate dense depth maps aggregated to a pointcloud. Make sure to update the data paths in the configuration file:

    .. code-block:: sh

        cd MonoRec/
        python3 create_pointcloud.py --config configs/test/pointcloud_monorec_tumvi.json

Visualization
-------------

1. Visualize the camera trajectory from Basalt:

    .. code-block:: sh

        cd trajectory_visualization/
        python3 visualize_trajectory.py <data_path>

2. Visualize the generated pointcloud using `CloudCompare <https://www.danielgm.net/cc/>`_ or `Open3D <http://www.open3d.org/>`_:

    .. code-block:: sh

        cloudcompare <pointcloud_path>

    or

    .. code-block:: sh

        python3 MonoRec/rgbd2pcl.py --config MonoRec/configs/test/pointcloud_monorec_euroc.json