.. _tuto01:

***************************************
[*Tuto01Basic*] Create an application
***************************************

The first tutorial represents a basic application that launches a simple empty frame. 

.. figure:: ../media/tuto01Basic.png
    :scale: 50
    :align: center
    

Prerequisites
--------------

Before reading this tutorial, you should have seen :
 * :ref:`Object-service concept<Object-Service_example>`
 * :ref:`App-config`
 * :ref:`Component`
 

Structure
----------

An application is organized around three main files : 
 * CMakeLists.txt
 * Properties.cmake
 * plugin.xml
 
CMakeLists.txt
~~~~~~~~~~~~~~~

The CMakeLists is parsed by CMake_. For the aplication it should contain the line : 

.. code::

    fwLoadProperties()

This line allows to load Properties.cmake file.

.. _CMake: https://cmake.org

Properties.cmake
~~~~~~~~~~~~~~~~~

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto01Basic ) # Name of the application
    set( VERSION 0.1 ) # Version of the application
    set( TYPE APP ) # Type APP represent "Application"
    set( DEPENDENCIES  ) # For an application we have no dependencies (libraries to link)
    set( REQUIREMENTS # The bundles used by this application
        dataReg # to load the data registry
        servicesReg # to load the service registry
        gui # to load gui
        guiQt # to load qt implementation of gui
        fwlauncher # executable to run the application
        appXml2 # to parse the application configuration
    )

    # Set the configuration to use : 'tutoBasicConfig'
    bundleParam(appXml2 PARAM_LIST config PARAM_VALUES tutoBasicConfig) 

    
This file contains the minimal requirements to launch an application with a Qt user interface.

.. note::

    The Properties.cmake file of the application is used by CMake_ to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application. 
    

plugin.xml
~~~~~~~~~~~

This file is located in the ``rc/`` directory of the application. It defines the services to run.
 
.. code-block:: xml

    <!-- Application name and version (the version is automatically replaced by CMake
         using the version defined in the Properties.cmake) -->
    <plugin id="Tuto01Basic" version="@DASH_VERSION@">

        <!-- The bundles in requirements are automatically started when this 
             Application is launched. -->
        <requirement id="dataReg" />
        <requirement id="servicesReg" />
        <requirement id="guiQt" />

        <!-- Defines the App-config -->
        <extension implements="::fwServices::registry::AppConfig2">
            <id>tutoBasicConfig</id><!-- identifier of the configuration -->
            <config>

                <!-- Frame service -->
                <service uid="myFrame" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>tutoBasicApplicationName</name>
                            <icon>@BUNDLE_PREFIX@/Tuto01Basic_0-1/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                    </gui>
                </service>

                <start uid="myFrame" /><!-- start the frame service -->

            </config>
        </extension>
    </plugin>


The ``::fwServices::registry::AppConfig2`` extension defines the configuration of an application. 

**id**: 
    The configuration identifier.
**config**: 
    Contains the list of objects and services used by the application. 
    
    For this tutorial, we have no object and only one service ``::gui::frame::DefaultFrame``.
    
    There are others tags that will be described in the next tutorials.


Run
----

To run the application, you must call the following line into the install or build directory:

.. code::

    bin/fwlauncher Bundles/Tuto01Basic_0-1/profile.xml
