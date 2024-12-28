Euroc Preparation
=================

The final directory structure for monorec to run on a euroc type dataset must be as shown below. 
The images must be from a camera-model of type *pinhole*. The number *00* represents the sequence name. The folders *dso* and *mav0* follow the euroc convention.  
*mav0* contains the data (images, imu etc.) and *dso* contains sensor information (camera intrinsics, camera extrinsics, distortion parameters, imu intrinsics etc.)

.. code-block:: text

    dataset-dir
        ├── 00
        |    ├── basalt_keyframe_data
        |    │   ├── keypoints
        |    │   ├── keypoints_viz
        |    │   └── poses
        |    ├── dso
        |    │   ├── cam0
        |    │   │   └── images -> ../../mav0/cam0/data
        |    │   └── cam1
        |    │       └── images -> ../../mav0/cam1/data
        |    ├── mav0
        |        ├── cam0
        |        │   └── data
        |        ├── cam1
        |        │   └── data
        |        ├── imu0
        |        └── mocap0
        ├── 01
        ...



.. toctree::
    :caption: Contents:
    :maxdepth: 2

    realsense_bag
    tumvi


The `run_basalt_euroc.py <https://github.com/hardik01shah/DenseReconstructionTools/blob/master/euroc-preparation/run_basalt_euroc.py>`_ script is used for generating
the *basalt_keyframe_data* folder in the above mentioned directory structure.
It runs basalt on each of the sequences and saves the keypoints and poses.

.. code-block:: text

    usage: run_basalt_euroc.py [-h] tumvi_path basalt_path

    Python code for running basalt on tumvi sequences

    positional arguments:
      tumvi_path   path to tum-vi dataset
      basalt_path  path to basalt directory

    optional arguments:
      -h, --help   show this help message and exit

    E.g.
    python3 run_basalt_tumvi.py ../../tumvi-dataset/ ../basalt/
