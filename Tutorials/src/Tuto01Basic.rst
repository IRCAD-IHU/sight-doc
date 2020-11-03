.. _TutorialsTuto01basic:

***************************************
[*Tuto01Basic*] Create an application
***************************************

The first tutorial represents a basic application that launches a simple empty frame. It introduces the concept of XML
application configuration and CMake generation.

.. figure:: ../media/tuto01Basic.png
    :scale: 25
    :align: center

=========
Structure
=========

Sight is organized around four elements: the ``application``, the ``module``, the ``library`` and the ``utility``.

The ``applications`` contain the configuration of the ``modules`` (and its services) to launch. The ``modules`` contain
the cpp implementation of the services, it may also contain some application sub-configuration. The ``libraries``
contain the data implementation and the code shared with several modules. The ``utilities`` are simple executables using
the ``libraries``.

In this example, we will only explain how to create a basic application with the existing modules. Further Tutorials
will explain how to use and create services and modules.

A sight application is organized around three main files :
 * CMakeLists.txt
 * Properties.cmake
 * plugin.xml

--------------
CMakeLists.txt
--------------

The CMakeLists.txt is parsed by CMake_. For an application, it should contain the following lines :

.. code-block:: cmake

    fwLoadProperties()

- ``fwLoadProperties()`` allows to load Properties.cmake file and thus to build the application

.. _CMake: https://cmake.org

----------------
Properties.cmake
----------------

This file describes the project information and requirements (see :ref:`HowTosCMakeProperties.cmake`) :

.. code-block:: cmake

    set( NAME Tuto01Basic )     # Name of the application
    set( VERSION 0.2 )          # Version of the application
    set( TYPE APP )             # Type APP represent "Application"
    set( DEPENDENCIES  )        # For an application we have no dependencies (libraries to link)
    set( REQUIREMENTS           # The modules used by this application
        fwlauncher              # Needed to build the launcher
        appXml                  # XML configurations
        guiQt                   # Start the module, load qt implementation of gui

        servicesReg             # fwService

        # UI declaration/Actions
        gui
    )

    moduleParam(
            appXml
        PARAM_LIST
            config
        PARAM_VALUES
            Tuto01Basic_AppCfg
    ) # Main application's configuration to launch

This file contains the minimal requirements to launch an application with a Qt user interface.

.. note::

    The Properties.cmake file of the application is used by CMake_ to compile the application but also to generate the
    ``profile.xml``, the input file used to launch the application (see :ref:`profile.xml`).

The ``moduleParam`` line defines the parameters to set for a module, here it defines the configuration to launch by the
appXML module, i.e. the application configuration.

----------
plugin.xml
----------

This file is located in the ``rc/`` directory of the application. It contains the application configuration.

.. code-block:: xml

    <!-- Application name and version (the version is automatically replaced by CMake
         using the version defined in the Properties.cmake) -->
    <plugin id="Tuto01Basic" version="@PROJECT_VERSION@" >

        <!-- The modules in requirements are automatically started when this
             Application is launched. -->
        <requirement id="servicesReg" />
        <requirement id="guiQt" />

        <!-- Defines the App-config -->
        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto01Basic_AppCfg</id><!-- identifier of the configuration -->
            <config>

                <!-- ******************************* UI declaration *********************************** -->

                <service uid="myFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto01Basic</name>
                            <icon>Tuto01Basic-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                    </gui>
                </service>

                <!-- ******************************* Start services ***************************************** -->

                <start uid="myFrame" />

            </config>
        </extension>
    </plugin>


``<requirement>`` lists the modules that should be loaded before launching the application:
the module to register data or i/o services (see Requirements_).

The ``::fwServices::registry::AppConfig`` extension defines the configuration of an application:

**id**:
    The configuration identifier.
**config**:
    Contains the list of objects and services used by the application.
    For this tutorial, we have no object and only one service ``::gui::frame::SDefaultFrame``.
    There are few others tags that will be described in the next tutorials.

.. _Requirements: https://sight.pages.ircad.fr/sight/group__requirement.html

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto01basic

   .. group-tab:: Windows

        .. code::

            bin/tuto01basic.bat
