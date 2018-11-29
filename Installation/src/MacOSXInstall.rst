Installation for MacOSX
=======================

Prerequisites
-----------------

If not already installed:

#. Install `Xcode <https://itunes.apple.com/fr/app/xcode/id497799835?mt=12>`_
#. Install `git <https://git-scm.com/downloads>`_
#. Install `CMake <http://www.cmake.org/download/>`_. The minimal version required is **3.7** if you want to compile with precompiled headers (build twice faster, enabled by default). Otherwise you can use a 3.1 version.
#. Install `Ninja <https://ninja-build.org/>`_ : to use instead of **make**.

For an easy install, you can use the `Hombrew project <http://brew.sh/>`_  to install missing packages.

.. code:: bash

    $ brew install git
    $ brew install cmake
    $ brew install ninja

For Openni dependency, you need libusb

.. code:: bash

    $ brew install libusb-compat

If you plan to build extra dependencies, the `VLC <https://www.videolan.org/vlc/index.fr.html>`_ application is also needed.

.. code:: bash

    $ brew cask install vlc

.. include:: CommonDeps.rst

Build
~~~~~~~~

Now you can compile the Sight dependencies with make in the console, it will automaticaly download, build and install each dependency.

.. code:: bash

    $ make install
    $ make install_tool

.. include:: CommonSrc.rst

Recommended software
--------------------------------

The following programs may be helpful for your developments:

- IDE:
    - `Qt creator <http://www.qt.io/download-open-source/#section-6>`_
    - `Eclipse CDT <https://eclipse.org/cdt/>`_.

- Versioning tools:
    - `SourceTree <http://www.sourcetreeapp.com/>`_
