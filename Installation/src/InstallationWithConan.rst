Installation with Conan (pre-built dependencies)
================================================

Introduction
------------

From `Conan documentation <https://docs.conan.io/en/latest/>`_ :

"Conan is a portable package manager, intended for C and C++ developers, but it is able to manage builds from source,
dependencies, and pre-compiled binaries for any language."

This means that by using Conan we are no longer required to build dependencies ourselves. By following these
instructions, you will be able to build sight with downloaded pre-compiled dependencies.

Note that all dependencies packages can be found in our very own
`Artifactory webpage <https://conan.ircad.fr/artifactory/webapp/#/home>`_

Prerequisites
-------------

.. tabs::

   .. group-tab:: Linux

        For now, only Linux Mint 18 and 19 distributions are supported, other distributions may work but have
        not been tested.

   .. group-tab:: Windows

        For now, only `Visual Studio 2017` is supported, other versions may work but have not been tested.


   .. group-tab:: Mac OSX

        For now, only `Mojave 10.14` is supported, other versions may work but have not been tested.

If not already installed:

#. Install `git <https://git-scm.com/>`_
#. Install `Python 3.5 or greater <https://www.python.org/downloads/>`_
#. Install `CMake <http://www.cmake.org/download/>`_ Version 3.12 or later is required. You should use prebuilt binaries as it is safer.
#. Install `ninja <https://github.com/ninja-build/ninja/releases>`_
#. Install `Conan  <https://docs.conan.io/en/latest/installation.html>`_ (you can use Python's ``pip``, but be sure to use python ``3`` -- you can check this by running ``pip --version``)

Install a c++ compiler and other development libraries.

.. tabs::

   .. group-tab:: Linux

        Install `gcc 7 <https://gcc.gnu.org/>`_  or `clang 6.0 <http://clang.llvm.org/>`_ (As pre-built packages are
        only compiled with this versions).

        Depending on which linux distribution you use,
        for example on **Mint**, you can run the following command to download and install these tools:

        .. code:: bash

            $ sudo apt-get install build-essential ninja-build python3 git

        And dependency development libraries :

        .. code:: bash

            $ sudo apt-get install libz3-dev libiconv-hook-dev libpng-dev libjpeg-turbo8-dev \
              libtiff5-dev libfreetype6-dev libxml2-dev libexpat1-dev libicu-dev libfontconfig1-dev \
              libx264-dev libx265-dev libusb-dev libusb-1.0-0-dev libxcb.*-dev  libx11-xcb-dev \
              libglu1-mesa-dev libglew-dev libvlc-dev libvlccore-dev vlc libxrender-dev libxi-dev \
              libasound2-dev libgstreamer1.0-dev libssl-dev libxt-dev libgstreamer-plugins-*1.0-dev \
              libudev-dev libxaw7-dev libxrandr-dev libavcodec-dev libavutil-dev libavformat-dev \
              libswscale-dev libavresample-dev

   .. group-tab:: Windows

        Install `Visual Studio 2017 Community <https://visualstudio.microsoft.com/>`_


   .. group-tab:: Mac OSX

        Install `Xcode 10.1 <https://itunes.apple.com/fr/app/xcode/id497799835?mt=12>`_

        For an easy install, you can use the `Homebrew project <http://brew.sh/>`_  to install missing packages.
        Brewed python is python ``3`` and is required since default macOS python is ``2``

        .. code:: bash

            $ brew install git
            & brew install python
            $ brew install cmake
            $ brew install ninja


Source tree layout
~~~~~~~~~~~~~~~~~~~

Good practices in Sight recommend to separate source files, build and install folders.
So to prepare the development environment:

* Create a development folder (Dev)

* Create a build folder (Dev/Build)

    * Add a sub folder for Debug and Release.

* Create a source folder (Dev/Src)

* Create an install folder (Dev/Install)

    * Add a sub folder for Debug and Release.

|directories|

Of course you can name the folders as you wish, or choose a different layout, but keep in mind to not build inside the
source directory. This is strongly discouraged by *CMake* authors.

.. |directories| image:: ../media/DirectoriesNoDeps.png


.. _settingUpEnv:

Setting up your environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tabs::

   .. group-tab:: Linux

        Make sure all of your Prerequisites_ are loaded into your path correctly, for all installation made through
        `apt-get` this is done automatically but for manually downloaded binaries (e.g. example) you'll need to use
        this command :

        .. code::

            $ PATH=$HOME/<cmake-bin-path>:$PATH

        .. tip::

            Adding this line to a start-up script can save you time and effort!

   .. group-tab:: Windows

        Load into your active PATH environment variable the needed locations in-order to be able to build.

        * Add Visual studio compilers.

        You can use the 'VS2017 x64 Native Tools Command Prompt'  or launch the `vcvarsall.bat` script with the parameter
        `amd64` on your current console.
        The location of that script will look something like this
        ``C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvarsall.bat``

        * Add the Prerequisites_

        If installed with default parameters ``git``, ``CMake`` and ``Python`` will be automatically loaded into your PATH
        variable.

        For static binaries like ``Ninja`` you will need to add them manually with a command similar to :

        .. code:: bash

            > PATH=%PATH%;C:\Bin

        .. tip::

            Writing a ``.bat`` script that loads all these previous locations to your path can save you time and effort!


   .. group-tab:: Mac OSX

        Make sure all of your Prerequisites_ are loaded into your path correctly, for all installation made through
        `brew` this is done automatically but for manually downloaded binaries you'll need to do it yourself.

        If you haven't done it, launch Xcode at least one time and install the ``Command Line Tools`` when prompted.

        You can do this manually by using the following command:

        .. code:: bash

            $ xcode-select --install

        If you already had installed the ``Command Line Tools``, it may be a good idea to check that the currently used ones are the default:

        .. code:: bash

            $ xcode-select --print-path
            /Applications/Xcode.app/Contents/Developer

        If the above command print something different, you mays reset to the default with:

        .. code:: bash

            $ sudo xcode-select --reset

Building your sources
----------------------

* `Clone <http://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository#Cloning-an-Existing-Repository>`_ the following repository in the (Dev/Src) source folder:

    * `sight <https://git.ircad.fr/Sight/sight.git>`_


.. code:: bash

    $ cd Dev/Src
    $ git clone https://git.ircad.fr/Sight/sight.git

* Go into your Build directory (Debug or Release) : here is an example if you want to compile in debug:

.. code:: bash

    $ cd Dev/Build/Debug

.. warning:: Make sure your environment is properly set : :ref:`settingUpEnv` .

* Call cmake-gui.

.. code:: bash

    $ cmake-gui .

Configuration
~~~~~~~~~~~~~~~~

* Set the desired Build directory (e.g. Dev/Build/Debug or Release)

* Set the desired Source directory (e.g. Dev/Src/sight)

* Click on "configure".

* During configure step, choose the generator 'Ninja' to compile Sight sources.

Generation
~~~~~~~~~~~~~~

* Set the following arguments:

``CMAKE_INSTALL_PREFIX``:
    Set the install location (e.g. Dev/Install/Debug).
``CMAKE_BUILD_TYPE``:
    Set to Debug or Release.
``PROJECTS_TO_BUILD``:
    Set the names of the applications to build (see Dev/Src/Apps or Dev/Src/Samples, ex: VRRender, Tuto01Basic ...),
    each project should be separated by ";".
``USE_CONAN``:
    This box ensures Conan packages are downloaded instead of relying on local builds. (check advanced options).

.. note::

    - If ``PROJECTS_TO_BUILD`` is empty, all application will be compiled.

* Click on "generate".

If you want to launch the ``cmake``  through the command line with the appropriate parameters

.. code:: bash

    $ cd Dev/Build/Debug
    $ cmake <path_to_sources> -G "Ninja" -DCMAKE_INSTALL_PREFIX=<Path_to_install_dir> \
      -DCMAKE_BUILD_TYPE=Debug -DUSE_CONAN=ON

Build
~~~~~~~

* Compile the Sight source using ninja in the console:

    * Go to the build directory (e.g. Dev/Build/Debug or Release)
    * Use "ninja" if you want to compile all the applications set in CMake.
    * Use "ninja name_of_application" to compile only one of the applications set in CMake.

.. code:: bash

    $ cd Dev/Build/Debug
    $ ninja

Launch an application
---------------------

After a successful compilation any previously built application can be launched with the appropriate script from Sight.

.. tabs::

   .. group-tab:: Linux

        You will find in the ``Build/bin`` directory an automatically generated script with the same name (on lowercase)
        as the application you built.

        .. code:: bash

            $ cd Dev/Build/Debug
            $ ./bin/myapplication



   .. group-tab:: Windows

        You will find in the ``Build\bin`` directory an automatically generated ``.bat`` with the same name (on
        lowercase) as the application you built.

        .. code:: bash

            $ cd Dev/Build/Debug
            $ ./bin/myapplication.bat


   .. group-tab:: Mac OSX

        You will find in the ``Build/bin`` directory an automatically generated script with the same name (on lowercase)
        as the application you built.

        .. code:: bash

            $ cd Dev/Build/Debug
            $ ./bin/myapplication

.. important::
    This automatically generated script loads all the needed Conan packages locations and adds them temporarily to your
    PATH variable. Feel free to take a look inside.

Generate an installer
---------------------

After setting the applications for which you want to generate installers in the ``PROJECTS_TO_BUILD`` CMake variable
and generating the code, follow these two steps:

    * Run *ninja install application_to_install* in the Build directory
    * Run *ninja package* in the Build directory

The installer will be generated in the Build directory.

.. note::

    This functionality is only fully supported on Windows and Linux distributions.

    For Mac OSX, ninja install will generate a ``.app`` and works only on some applications.

Recommended software
--------------------

The following programs may be helpful for your developments:

.. tabs::

   .. group-tab:: Linux

        * `QT Creator <https://download.qt.io/official_releases/qtcreator/>`_:
            QT Creator is a multi-OS Integrated Development Environment (IDE) for computer programming.
            You can find a setup tutorial here :ref:`qtcreatorsetup`.

   .. group-tab:: Windows

        * `QT Creator <https://download.qt.io/official_releases/qtcreator/>`_:
            QT Creator is a multi-OS Integrated Development Environment (IDE) for computer programming.
            You can find a setup tutorial here :ref:`qtcreatorsetup`.
        * `Notepad++ <http://notepad-plus-plus.org/>`_:
            Notepad++ is a free source code editor, which is designed with syntax highlighting functionality.
        * `ConsoleZ <https://github.com/cbucher/console/wiki/Downloads>`_:
            ConsoleZ is an alternative command prompt for Windows, adding more capabilities to the default Windows command
            prompt. To compile Sight with the console the windows command prompt has to be set in the tab settings.

   .. group-tab:: Mac OSX

        * `QT Creator <https://download.qt.io/official_releases/qtcreator/>`_:
            QT Creator is a multi-OS Integrated Development Environment (IDE) for computer programming.
            You can find a setup tutorial here :ref:`qtcreatorsetup`.


Need some help? Keep in touch!
-------------------------------

As any active community, we *sighters* are happy to help each other or beginners however we can. Feel free to join us
and share with us your questions or comments at our `Gitter <https://gitter.im/IRCAD-IHU/sight-support>`_ .
We provide support in French, English and Spanish.
