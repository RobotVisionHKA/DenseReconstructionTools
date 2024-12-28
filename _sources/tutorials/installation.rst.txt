Installation
============

Installation of the packages required for the project is detailed below and must be followed in the order specified.

Dense Reconstruction Toolkit
----------------------------

1. Clone the repository:

    .. code-block:: bash

        git clone https://github.com/RobotVisionHKA/DenseReconstructionTools.git
        cd DenseReconstructionTools

2. Clone the submodules:

    .. code-block:: bash

        git submodule update --init --recursive

3. Install the required dependencies:

    .. code-block:: bash

        conda env create -f environment.yml
        conda activate dense_reconstruction_toolkit

Basalt
------

.. code-block:: bash

    cd basalt
    ./scripts/install_deps.sh
    mkdir build
    cd build
    cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
    make -j8

MonoRec
-------

The `conda` environment for this project must be setup by running the following command:

.. code-block:: bash

    conda env create -f MonoRec/environment.yml

