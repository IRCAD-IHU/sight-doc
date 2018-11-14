******************************
How to add a new Dependency ?
******************************

Sight dependencies are based on the `ExternalProject <http://www.cmake.org/cmake/help/v3.0/module/ExternalProject.html>`_ concept.

The concept is to create custom targets to build projects in external trees.
Each project has custom steps for download, update/patch, configure, build and install.

You may want to add a new dependency into sight-deps or you may want to add your own folder of dependencies.

.. tip::
    You need to know that the main CMakeLists.txt is in sight-deps, and you can add as many additional folders as you want.
    Use the ``ADDITONNAL_DEPS`` option in cmake to set the path of your custom dependency.

Add a new dependency in sight-deps
------------------------------------

Adding a new dependency is quite easy, the only things to do is to add a new folder *myNewDeps* and put a CMakeLists.txt
file into it. And you also have to add a add_subdirectory directive in the parent CMakeLists.txt.

The CMakeLists.txt should contain at least:

- cmake_minimum_required(...)
- project(...)
- include(ExternalProject)
- ExternalProject_Add(...)

Here is a simple example from glm :

.. code-block:: cmake

    cmake_minimum_required(VERSION 3.0)

    project(glmBuilder)

    include(ExternalProject)

    set(GLM_CMAKE_ARGS ${COMMON_CMAKE_ARGS}
                       -DBUILD_DOXYGEN:BOOL=OFF
    )

    set(CACHED_URL https://github.com/g-truc/glm/archive/0.9.8.4.tar.gz)

    ExternalProject_Add(
        glm
        URL ${CACHED_URL}
        URL_HASH SHA256=a220e60f8711265595be3221e530d632d5823641ecd46a3a54bc174933bff14c
        DOWNLOAD_DIR ${ARCHIVE_DIR}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
        CMAKE_ARGS ${GLM_CMAKE_ARGS}
    )

The important parts are in the *ExternalProject_Add* function:

- URL: is the download link of the sources
- DOWNLOAD_DIR: The folder where the sources will be stored (set globally for all dependencies)
- DEPENDS: The dependencies of the current library (will be compiled before)
- INSTALL_DIR: The folder in which the library will be installed (set globally for all dependencies)
- CMAKE_ARGS: CMake options for library which have a cmake build system

.. note::

    Note that in other script you can have much more options like:

    - PATCH_COMMAND
    - CONFIGURE_COMMAND
    - BUILD_COMMAND
    - INSTALL_COMMAND

    Refer you to the documentation of `ExternalProject <http://www.cmake.org/cmake/help/v3.0/module/ExternalProject.html>`_ for more informations.


Add a custom dependency repository
------------------------------------

You may want to add your own folder of dependencies. In this case your main need to create a
CMakeLists.txt in the root of your folder (myDepsFolder/CMakeLists.txt) in order to list the subdirectories of your dependencies.

.. code-block:: cmake

    cmake_minimum_required(VERSION 3.1)

    project(CustomDeps)

    list(APPEND SUBDIRECTORIES myDeps1)
    list(APPEND SUBDIRECTORIES myDeps2)
    # ...

Then when you do a *ccmake* or *cmake-gui* in the build of your dependency, you need to add the path to your custom
repository in the ``ADDITONNAL_DEPS`` option. Then cmake will automatically parse your folder.
