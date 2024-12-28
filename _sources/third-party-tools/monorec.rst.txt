MonoRec - Dense Reconstruction
==============================

This is a clone of the `MonoRec <https://github.com/Brummi/MonoRec>`_ repository with changes 
to run inference and train on Euroc type datasets, specifically the 
`TUM-VI <https://vision.in.tum.de/data/datasets/visual-inertial-dataset>`_ dataset.  

The tum-vi dataset is a visual inertial dataset that contains sequences recorded from a handheld 
setup consisting of a stereo setup with two cameras (fisheye lens). The images are grayscale. 
It also provides synchronized IMU data (gyro and accel).  

The primary additions are:  

1. Custom dataloader for tum-vi/euroc-format datasets  
2. Alternate script for viewing pointclouds using the 
   `Open3D <http://www.open3d.org/docs/latest/index.html>`_ library  

TUM-VI dataloader
-----------------

The `tum-vi dataloader <https://github.com/RobotVisionHKA/MonoRec/blob/main/data_loader/tum_vi_dataset.py>`_ has been written in a way so that 
it expects the dataset to be in a specific format as shown below:  

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

The overall pipeline of dataloading goes as follows:

1. Load camera intrinsics for each sequence  
   
2. Format the intrinsics according to the target image size  
   
3. Load the poses, left stereo images, right stereo images and sparse depth keypoints
  
    - The primary key is the poses i.e. only those timestamps for which keyframe pose is available is included in the dataset  
    - Poses are loaded and stored directly in the memory on initialization  
    - Stereo images and keypoints paths are stored on initialization and are accessed from the memory only during the ``_get_item()`` call  

4. Accessing images:  
   
    - convert to 3-channel image
    - image is first resized (if applicable) and then cropped to the target image size  
  
5. Accessing keypoints:  
   
    - ``.txt`` file containing the keypoints is read  
    - check for invalid entry i.e. nans or index out of bounds of the original image size  
    - scale the keypoints according to the target image size and add to depth tensor  
    - crop the depth tensor to target image size  

.. note::

   python dictionaries have been used for the above implementation. Good references for effective dataloader implementations [`ref1 <https://discuss.pytorch.org/t/how-to-prefetch-data-when-processing-with-gpu/548/19>`_] [`ref2 <https://discuss.pytorch.org/t/problem-with-dataloader-when-using-list-of-dicts/67268/4>`_]

PointCloud Visualization using open3d
-------------------------------------

The `rgbd2pcl.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/rgbd2pcl.py>`_ script is used to generate and view pointclouds from the keyframe, 
predicted depth, camera intrinsics and extrinsics. It also saves the keyframes and the predicted depth maps 
in the save directory mentioned in the config file (can be used for debugging). It uses Open3d for the same. 
[`ref1 <http://www.open3d.org/docs/latest/tutorial/Advanced/multiway_registration.html#Make-a-combined-point-cloud>`_] 
[`ref2 <http://www.open3d.org/docs/latest/tutorial/Basic/rgbd_image.html>`_]  

Make sure to activate the conda environment (monorec with open3d installation):  

.. code-block:: bash

    conda activate pcl

E.g.

.. code-block:: bash

    python3 rgbd2pcl.py --config configs/test/pointcloud_monorec_euroc.json

Inference
---------

The `example-tumvi <https://github.com/RobotVisionHKA/MonoRec/blob/main/example-tumvi>`_ folder can be used to test the forward pass using the tum-vi dataloader. 
The `test_monorec.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/example-tumvi/test_monorec.py>`_ script can be used to 
test inference on an entire dataset i.e. with multiple sequences, and 
the `test_monorec_seq.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/example-tumvi/test_monorec_seq.py>`_ 
can be used to test inference on a single sequence. 
  
Make sure to activate the conda environment for both inference and training using:  

.. code-block:: bash

    conda activate monorec

Usage:  

.. code-block:: bash

    python3 test_monorec.py

**_set pretrain_mode=1 to just evaluate the depth module without using the mask module_**

Pointcloud generation
---------------------

To evaluate the model, a pointcloud can be generated. `CloudCompare <https://www.danielgm.net/cc/>`_ was used for 
viewing the generated pointclouds. Either `rgbd2pcl.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/rgbd2pcl.py>`_ or 
`create_pointcloud.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/create_pointcloud.py>`_ can be used. 
Usage of `rgbd2pcl.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/rgbd2pcl.py>`_ is mentioned above.  

Usage for `create_pointcloud.py <https://github.com/RobotVisionHKA/MonoRec/blob/main/create_pointcloud.py>`_:  

.. code-block:: bash

    python create_pointcloud.py --config configs/test/pointcloud_monorec_tumvi.json

Training
--------

.. note::

   Change Ubuntu GUI mode for better speed during training* [`ref1 <https://linuxconfig.org/how-to-disable-enable-gui-on-boot-in-ubuntu-20-04-focal-fossa-linux-desktop>`_] [`ref2 <https://medium.com/@leicao.me/how-to-run-xorg-server-on-integrated-gpu-c5f38ae7ccc8>`_]   
   Good practices for training on multiple GPUs [`ref <https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255>`_]

Run the following commands:  

.. code-block:: bash

    python train.py --config configs/train/monorec/monorec_depth_tumvi.json --options stereo                          # Depth Bootstrap
    python train_monorec.py --config configs/train/monorec/monorec_mask_tumvi.json --options stereo                   # Mask Bootstrap
    python train_monorec.py --config configs/train/monorec/monorec_mask_ref_tumvi.json --options mask_loss            # Mask Refinement
    python train_monorec.py --config configs/train/monorec/monorec_depth_ref_tumvi.json --options stereo stereo_repr  # Depth Refinement

To monitor the training using tensorboard, set the parameter ``tensorboard`` to ``true`` in the config, and run the command below in a separate terminal:  

.. code-block:: bash

    MonoRec$ tensorboard --logdir=saved/log/monorec_depth/00

Important Hyperparameters for TUM-VI/RealSense-Bag
--------------------------------------------------

Some hyperparameters needed to be tuned differently for the TUM-VI dataset or the dataset recorded using the RealSense from the ones used in the paper for the KITTI dataset:

1. The ``inv_depth_min_max`` parameter must be set to (1.0, 0.0025) for training as the dataset has been recorded using a hand-held device as opposed to a device mounted on a car (KITTI).  
2. The ``step_size`` and ``gamma`` parameters of the ``lr_scheduler`` must be properly tuned keeping in mind the size of the dataset.  
3. The parameter ``alpha`` which is responsible for assigning weight to the ``sparse_depth_loss`` and the ``self_supervision_loss`` (combination of photometric_inconsistency_cv and edge_aware_smoothness_loss) must be set properly after observing the intermediate results during training.  
4. The ``num_workers`` and ``batch_size`` parameters must be set considering the compute power, size of dataset etc. [`ref1 <https://chtalhaanwar.medium.com/pytorch-num-workers-a-tip-for-speedy-training-ed127d825db7>`_] [`ref2 <https://deeplizard.com/learn/video/kWVgvsejXsE>`_]