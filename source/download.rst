.. diffusionkit documentation master file, created by
   sphinx-quickstart on Mon Oct 26 10:48:55 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

====================
Download and Install
====================

Download Links
==============

.. |winlogo| image:: images/winlogo.png
.. |rellogo| image:: images/rellogo.png
.. |linuxlogo| image:: images/linuxlogo.png

Latest Release (v1.2-r160310)
------------------------------

.. sidebar:: Previous Release

 See `Releases <https://github.com/liangfu/diffusionkit/releases>`_ for our previously compiled packages.

|winlogo| `Windows Installer (x86-64) 
<https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/DiffusionKitSetup-WIN64-v1.2-r160310.exe>`_ [11.9 MB]

|linuxlogo| `Linux Binary Package (x86-64) 
<https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz>`_ [36.4 MB]

Example Data and Test Script
----------------------------

* `List File <https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/list.txt>`_ [12 Bytes]
* `Subject 01 <https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/sub01.tar.gz>`_ [74.5 MB]
* `Subject 02 <https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/sub02.tar.gz>`_ [72.9 MB]
* `Brain Atlas <https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/atlas.tar.gz>`_ [3.33 MB]
* `Test Script <https://raw.githubusercontent.com/liangfu/diffusionkit/master/source/static/process_advanced.sh>`_ [1.8 KB]

And this is what we have in the data above::

 ├── atlas
 │   ├── aal.nii.gz
 │   ├── aal.nii.lut
 │   ├── aal.nii.txt
 │   ├── aal_roi_024.txt
 │   ├── aal_roi_090.txt
 │   └── ch2bet.nii.gz
 ├── atlas.tar.gz
 ├── list.txt
 ├── process_advanced.sh
 ├── process_primary.sh
 ├── sub01
 │   ├── dwi.bval
 │   ├── dwi.bvec
 │   ├── dwi.nii.gz
 │   └── t1.nii.gz
 ├── sub01.tar.gz
 ├── sub02
 │   ├── dwi.bval
 │   ├── dwi.bvec
 │   ├── dwi.nii.gz
 │   └── t1.nii.gz
 └── sub02.tar.gz

We recommand users to follow the `Tutorial Page <tutorials.html>`_ for 
a step-by-step introduction of the functions within DiffutionKit.

IN A HURRY? Download all the data files above and run the 
`Test Script <https://raw.githubusercontent.com/liangfu/diffusionkit/master/source/static/process_advanced.sh>`_ ,
or simply run the following commands to do everything. 

.. toggle_table::
  :arg1: Worldwide
  :arg2: China

.. toggle:: Worldwide

  .. code-block:: bash
   
    # install the program  
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz
    tar zxvf DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz
    export PATH=$PATH:`pwd`/DiffusionKitSetup-x86_64-v1.2-r160310/bin
  
    # get the data and run!
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/list.txt
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/sub01.tar.gz
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/sub02.tar.gz
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/atlas.tar.gz
    wget https://raw.githubusercontent.com/liangfu/diffusionkit/master/source/static/process_advanced.sh
    chmod +x process_advanced.sh
    ./process_advanced.sh	

.. toggle:: China

  .. code-block:: bash
   
    # install the program  
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz
    tar zxvf DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz
    export PATH=$PATH:`pwd`/DiffusionKitSetup-x86_64-v1.2-r160310/bin
  
    # get the data and run!
    wget https://github.com/liangfu/diffusionkit/releases/download/v1.2-r160310/list.txt
    wget http://ddl.escience.cn/ff/emBl?func=download -O sub01.tar.gz
    wget http://ddl.escience.cn/ff/emBm?func=download -O sub02.tar.gz
    wget http://ddl.escience.cn/ff/emBo?func=download -O atlas.tar.gz
    wget https://raw.githubusercontent.com/liangfu/diffusionkit/master/source/static/process_advanced.sh
    chmod +x process_advanced.sh
    ./process.sh	

And more simpler, download and run
`test-diffusionkit.sh <http://diffusion.brainnetome.org/en/latest/_static/test-diffusionkit.sh>`_
to take a quick look how DiffusionKit can **BUILD A BRAIN NETWORK JUST A FEW MINUTES**!

.. code-block:: bash

  $ bash test-diffusionkit.sh

System requirement
==================

Basically this software can run in any system, including 32/64-bit MS Windows/Linux OS, 
although currently we only tested and released the binary packages for Windows/Linux OS. 
The software is developed based on C/C++, and some platform independent packages, 
including VTK, and OpenCV, etc. 
However, for high-performance data processing and visualization, 
we recommend using 64-bit OS with multi-core CPU and standalone graphics card.

Install/Uninstall
=================

Please download the package from http://diffusion.brainnetome.org , 
according to your own OS. Unpack the files to where you want and you can enjoy the software. 
The 64-bit OS is recommended for high-performance data processing. 
Each installation package is completely standalone so you DO NOT need to 
install ANY other dependency to run the software. 
If you encounter any dependency problem please DO `contact us <mailto:diffusion.kit@nlpr.ia.ac.cn>`_.

For MS Windows OS
-----------------

Double click the ``DiffusionKitSetup-WIN64-v1.2-r160310.exe`` file and then choose the destination path 
according to the wizard. You may need to provide administrator permission 
if you want to put the files into the system path. 
Similarly, to uninstall you only need to hit the menu of “uninstall” in the MS Windows start menu.

For Linux OS
------------

:code:`Glibc>=2.2` is required. Download the ``DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz``, and then

.. code-block:: bash

 tar zxvf DiffusionKitSetup-x86_64-v1.2-r160310.tar.gz
 export PATH=$PATH:`pwd`/DiffusionKitSetup-x86_64-v1.2-r160310/bin

You could add the path to the $PATH in the ~/.bashrc file, by adding the following line,

.. code-block:: bash

 export $PATH=$PATH:/your/path/to/diffusionkit

To uninstall the software, just manually remove the entire folder where you untar-ed the .tar.gz file.

.. include:: common.txt




