.. _TutorialsTuto02DataServiceBasic:

*******************************************
[*Tuto02DataServiceBasic*] Display an image
*******************************************

The second tutorial represents a basic application that displays a medical 3D image.

.. figure:: ../media/tuto02DataServiceBasic.png
    :scale: 25
    :align: center

=============
Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`TutorialsTuto01Basic`

=========
Structure
=========

----------------
Properties.cmake
----------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto02DataServiceBasic )
    set( VERSION 0.2 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        fwlauncher              # Needed to build the launcher
        appXml                  # XML configurations
        guiQt                   # Start the module, load qt implementation of gui

        # Objects declaration
        fwData
        servicesReg             # fwService

        # UI declaration/Actions
        gui

        # Reader
        ioVTK                   # contains the reader and writer for VTK files (image and mesh).

        # Services
        visuBasic               # loads basic rendering services for images and meshes.
    )

    moduleParam(
            appXml
        PARAM_LIST
            config
        PARAM_VALUES
            TutoDataServiceBasic_AppCfg
    ) # Main application's configuration to launch

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``, the input file used to launch the application (see :ref:`profile.xml`).

----------
plugin.xml
----------

This file is located in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto02DataServiceBasic" version="@PROJECT_VERSION@" >

        <!-- The modules in requirements are automatically started when this
             Application is launched. -->
        <requirement id="servicesReg" />
        <requirement id="guiQt" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>TutoDataServiceBasic_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <!-- In tutoDataServiceBasic, the central data object is a ::fwData::Image. -->
                <object uid="imageData" type="::fwData::Image" />

                <!-- ******************************* UI declaration *********************************** -->

                <!--
                    Description service of the GUI:
                    The ::gui::frame::SDefaultFrame service automatically positions the various
                    containers in the application main window.
                    Here, it declares a container for the 3D rendering service.
                -->
                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto02DataServiceBasic</name>
                            <icon>Tuto02DataServiceBasic-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                    </gui>
                    <registry>
                        <!-- Associate the container for the rendering service. -->
                        <view sid="imageRendereSrv" />
                    </registry>
                </service>

                <!-- ******************************* Services ***************************************** -->

                <!--
                    Reading service:
                    The <file> tag defines the path of the image to load. Here, it is a relative
                    path from the repository in which you launch the application.
                -->
                <service uid="imageReaderSrv" type="::ioVTK::SImageReader" >
                    <inout key="data" uid="imageData" />
                    <file>../../data/patient1.vtk</file>
                </service>

                <!--
                    Visualization service of a 3D medical image:
                    This service will render the 3D image.
                -->
                <service uid="imageRendereSrv" type="::visuBasic::SImage" >
                    <in key="image" uid="imageData" />
                </service>

                <!-- ******************************* Start services ***************************************** -->

                <!--
                    Definition of the starting order of the different services:
                    The frame defines the 3D scene container, so it must be started first.
                    The services will be stopped the reverse order compared to the starting one.
                -->
                <start uid="mainFrame" />
                <start uid="imageReaderSrv" />
                <start uid="imageRendereSrv" />

                <!-- ******************************* Update services ***************************************** -->

                <!--
                    Definition of the service to update:
                    The reading service load the data on the update.
                    The render update must be called after the reading of the image.
                -->
                <update uid="imageReaderSrv" />
                <update uid="imageRendereSrv" />

            </config>
        </extension>

    </plugin>

For this tutorial, we have only one object ``::fwData::Image`` and three services:
 * ``::gui::frame::SDefaultFrame``: frame service
 * ``::ioVTK::SImageReader``: reader for 3D VTK image
 * ``::visuBasic::SImage``: renderer for 3D image

The following order of the configuration elements must be respected:
  #. ``<object>``
  #. ``<service>``
  #. ``<connect>`` (see :ref:`TutorialsTuto04SignalSlot`)
  #. ``<start>``
  #. ``<update>``

.. note::
    To avoid the ``<start uid="imageRendereSrv" />``, the frame service can automatically start the rendering service: you
    just need to add the attribute ``start="yes"`` in the ``<view>`` tag.

===
Run
===

To run the application, you must call the following line in the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto02dataservicebasic

   .. group-tab:: Windows

        .. code::

            bin/tuto02dataservicebasic.bat

