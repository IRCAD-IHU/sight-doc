.. _HowCMake:

How to use CMake with sight ?
===============================

Introduction
-------------

Sight and its dependencies are built with `CMake <http://www.cmake.org/>`_ .
Note that the minimal version of cmake to have is 3.9.


Each project (apps, bundles, libs) has two "CMake" files:

- CMakeLists.txt_
- Properties.cmake_

.. _CMakeLists.txt:

CMakeLists.txt
---------------

The *CMakeLists.txt* should contain at least the function *fwLoadProperties()* to load the Properties.cmake.
But it can also contain others functions useful to link with external libraries.

Here is an example of CMakeLists.txt from guiQt Bundle :

.. code-block:: cmake

    fwLoadProperties()

    find_package(Qt5 COMPONENTS Core Gui Widgets REQUIRED)


    fwInclude(
        ${Qt5Core_INCLUDE_DIRS}
        ${Qt5Gui_INCLUDE_DIRS}
        ${Qt5Widgets_INCLUDE_DIRS}
    )

    fwLink(
           ${Qt5Core_LIBRARIES}
           ${Qt5Gui_LIBRARIES}
           ${Qt5Widgets_LIBRARIES}
    )

    set_target_properties(${FWPROJECT_NAME} PROPERTIES AUTOMOC TRUE)

The first line *fwLoadProperties()* will load the *Properties.cmake* (see explanation in the next section).

The next lines allows to compile with the support of some external libraries (sight-deps), in this example this is Qt.
The first thing to do is to discover where Qt is installed. This is done with the regular CMake command ``find_package(The_lib COMPONENTS The_component)``.
Then we use ``fwInclude`` to add includes directories to the target, and ``fwLink`` to link the libraries with your target.
Actually if you're accustomed to CMake these two macros are strictly equivalent to:

.. code-block:: cmake

    target_include_directories( ${FWPROJECT_NAME} SYSTEM PRIVATE ${Qt5Core_INCLUDE_DIRS} ${Qt5Gui_INCLUDE_DIRS} ${Qt5Widgets_INCLUDE_DIRS} )
    target_link_libraries( ${FWPROJECT_NAME} PRIVATE ${Qt5Core_LIBRARIES} ${Qt5Gui_LIBRARIES} ${Qt5Widgets_LIBRARIES} )

They are proposed as a convenience so people won't forget for instance to specify `SYSTEM`,
which prevents compilation warnings from third-part libraries to be displayed.
If the rare case where your bundle may be a dependency of an another one,
you can forward the include directories and the libraries with ``fwForwardInclude`` and ``fwForwardLink``,
which are still equivalent to ``target_include_directories``
and ``target_link_libraries`` CMake commands but with ``PUBLIC`` set instead of ``PRIVATE``.

Eventually, you can also add custom properties to your target with ``set_target_properties``.

.. _Properties.cmake:

Properties.cmake
-----------------

Properties.cmake should contain informations like name, version, dependencies and requirements of the current target.

Here is an example of Properties.cmake from ``fwData`` library:

.. code-block:: cmake

 set( NAME fwData )
 set( VERSION 0.1 )
 set( TYPE LIBRARY )
 set( DEPENDENCIES fwCom fwMemory fwTools )
 set( REQUIREMENTS  )

NAME:
    Name of the target

VERSION:
    Version of the target

TYPE:
    Define the type of the target:

    - APP for an "Application"
    - BUNDLE for a "bundle"
    - LIBRARY for a "library"
    - EXECUTABLE for an executable

DEPENDENCIES:
    Link the target with the given libraries (see `target_link_libraries <http://www.cmake.org/cmake/help/v3.0/command/target_link_libraries.html?highlight=target_link_libraries>`_ ).
    The DEPENDENCIES should contain only "library".

REQUIREMENTS:
    Ensure that the dependencies are built before the targets (see `add_dependencies <http://www.cmake.org/cmake/help/v3.0/command/add_dependencies.html?highlight=add_dependencies>`_ ).
    The REQUIREMENTS should contain only "bundles".

In some Properties.cmake (mostly in applications), you can see the line:

.. code-block:: cmake

    bundleParam(appXml PARAM_LIST config PARAM_VALUES tutoBasicConfig)

This CMake macro allows to give parameters to a bundle. The parameters are defined like:

.. code-block:: cmake

    bundleParam(<bundle>
                PARAM_LIST <param1_name> <param2_name> <param3_name>
                PARAM_VALUES <param1_value> <param2_value> <param3_value>
                )

These parameters can be retrieved in the ``Plugin.cpp`` like:

.. code-block:: cpp

    void Plugin::start()
    {
        if( this->getBundle()->hasParameter("param1_name") )
        {
            const std::string param1Value = this->getBundle()->getParameterValue("param1_name");
        }
        if( this->getBundle()->hasParameter("param2_name") )
        {
            const std::string param2Value = this->getBundle()->getParameterValue("param2_name");
        }
        // ...
    }

For the application, this macro defines the main configuration to launch when the application is started.
