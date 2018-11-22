.. _tuto01:

***************************************
[*Tuto01Basic*] Create an application
***************************************

The first tutorial represents a basic application that launches a simple empty frame. It introduces the concept of XML 
application configuration and CMake generation.


.. figure:: ../media/tuto01Basic.png
    :scale: 50
    :align: center
    

Prerequisites
--------------

You should have properly installed your sight environment (see :ref:`Installation`).
 

Structure
----------

Sight is organized around four elements: the ``application``, the ``bundle``, the ``library`` and the ``utility``.

The ``applications`` contain the configuration of the ``bundles`` (and its services) to launch. The ``bundles`` contain
the cpp implementation of the services, it may also contain some application sub-configuration. The ``libraries`` 
contain the data implementation and the code shared with several bundles. The ``utilities`` are simple executables using 
the ``libraries``.

In this example, we will only explain how to create a basic application with the existing bundles. Further Tutorials 
will explain how to use and create services and bundles.

A sight application is organized around three main files : 
 * CMakeLists.txt
 * Properties.cmake
 * plugin.xml
 
CMakeLists.txt
~~~~~~~~~~~~~~~

The CMakeLists.txt is parsed by CMake_. For an application, it should contain the following lines : 

.. code-block:: cmake

    fwLoadProperties() 
    generic_install()

- ``fwLoadProperties()`` allows to load Properties.cmake file and thus to build the application
- ``generic_install()`` allows to generate an installer for the application

.. _CMake: https://cmake.org

Properties.cmake
~~~~~~~~~~~~~~~~~

This file describes the project information and requirements (see :ref:`Properties.cmake`) :

.. code-block:: cmake

    set( NAME Tuto01Basic )
    set( VERSION 0.1 )
    set( TYPE APP ) 
    set( DEPENDENCIES  )
    set( REQUIREMENTS 
        dataReg # to load the data registry
        servicesReg # to load the service registry
        gui # to load gui
        guiQt # to load qt implementation of gui
        fwlauncher # executable to run the application
        appXml # to parse the application configuration
    )

    # Set application configuration to 'tutoBasicConfig'
    bundleParam(appXml PARAM_LIST config PARAM_VALUES tutoBasicConfig) 

    
This file contains the minimal requirements to launch an application with a Qt user interface.

.. note::

    The Properties.cmake file of the application is used by CMake_ to compile the application but also to generate the
    ``profile.xml``, the input file used to launch the application (see :ref:`profile.xml`). 
    
The ``bundleParam`` line defines the parameters to set for a bundle, here it defines the configuration to launch by the 
appXML bundle, i.e. the application configuration.

plugin.xml
~~~~~~~~~~~

This file is located in the ``rc/`` directory of the application. It contains the application configuration.
 
.. code-block:: xml

    <!-- Application name and version (the version is automatically replaced by CMake
         using the version defined in the Properties.cmake) -->
    <plugin id="Tuto01Basic" version="@PROJECT_VERSION@">

        <!-- The bundles in requirements are automatically started when this 
             Application is launched. -->
        <requirement id="dataReg" />
        <requirement id="servicesReg" />

        <!-- Defines the App-config -->
        <extension implements="::fwServices::registry::AppConfig">
            <id>tutoBasicConfig</id><!-- identifier of the configuration -->
            <config>

                <!-- Frame service -->
                <service uid="myFrame" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>tutoBasicApplicationName</name>
                            <icon>Tuto01Basic-0.1/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                    </gui>
                </service>

                <start uid="myFrame" /><!-- start the frame service -->

            </config>
        </extension>
    </plugin>

``<requirement>`` lists the bundles that should be loaded before launching the application: the bundle to register data or 
i/o services (see Requirements_).

The ``::fwServices::registry::AppConfig`` extension defines the configuration of an application: 

**id**: 
    The configuration identifier.
**config**: 
    Contains the list of objects and services used by the application.     
    For this tutorial, we have no object and only one service ``::gui::frame::SDefaultFrame``.    
    There are few others tags that will be described in the next tutorials.

.. _Requirements: https://sight.pages.ircad.fr/sight/group__requirement.html

Run
----

To run the application, you must call the following line into the install or build directory:

.. code::

    bin/fwlauncher share/Tuto01Basic-0.1/profile.xml

On Linux and MacOs, you can also use the shortcut (generated for each application):

.. code::

    bin/tuto01basic
