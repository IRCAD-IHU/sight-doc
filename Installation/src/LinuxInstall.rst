Installation for Linux
======================

Prerequisites for Linux users
--------------------------------

If not already installed:

#. Install `git <https://git-scm.com/>`_
#. Install `gcc <https://gcc.gnu.org/>`_ The minimal version required is 4.8 or `clang <http://clang.llvm.org/>`_ The minimal version required is 3.5
#. Install `Python 2.7 <https://www.python.org/downloads/>`_
#. Install `CMake <http://www.cmake.org/download/>`_ The minimal version required is 3.1
#. Install `Ninja <https://ninja-build.org/>`_

Depending on which linux distribution you use, for example on Debian/Ubuntu/Mint you can do:

.. code:: bash

    $ apt-get install build-essential ninja-build python2.7 git cmake

.. warning::
    If the **CMake** version of your distribution is not sufficient (Mint 17 for instance ships only the 2.18 version), you can easily grab it on the `Kitware website <https://cmake.org/download/>`_. Download the **binary** version (much easier than compiling yourself), extract it to a folder (i.e. /home/login/software/cmake/) and add the ``bin/`` folder inside it to your ``PATH`` environment variable :

    .. code:: bash

        # ~/.bashrc
        export PATH=/home/login/software/cmake/bin:$PATH
    
Qt is an external library used in FW4SPL. For the successful compilation of Qt with FW4SPL, please see the following requirements:

- http://wiki.qt.io/Building_Qt_5_from_Git

Follow the instructions there to install the necessary packages on your system for **Build essentials**, **libxcb** and **QtMultimedia**. For the latter, please note that we use gstreamer-1.0 by default, so please replace ``libgstreamer0.10-dev`` and ``libgstreamer-plugins-base0.10-dev`` by ``libgstreamer1.0-dev`` and ``libgstreamer-plugins-base1.0-dev``. You can safely ignore instructions for QtWebKit and QtWebEngine, we don't build them.
Since we build Qt with openssl support you also need to install ``libssl-dev`` (be sure that the version is equal or upper to 1.0.0). Last for VTK we also need the X Toolkit Intrinsics library headers, that you can easily install for instance on a Debian-based distribution with the package ``libxt-dev``.

.. include:: CommonDeps.rst

Compilation
++++++++++++++

Now you can compile the FW4SPL dependencies with make in the console, it will automaticaly download, build and install each dependency.

.. code:: bash

    # Adjust the number of cores depending of the CPU cores and the RAM available on your computer
    $ make -j4 
    
.. include:: CommonSrc.rst

Recommended software
-------------------------

The following programs may be helpful for your developments:

- `Eclipse CDT <https://eclipse.org/cdt/>`_

- `QtCreator <https://www.qt.io/download-open-source/#section-2>`_

