.. _tuto02:

*********************************************
[*Tuto02DataServiceBasic*] Display an image
*********************************************

The second tutorial represents a basic application that displays a medical 3D image.

.. figure:: ../media/tuto02DataServiceBasic.png
    :scale: 50
    :align: center


Prerequisites
--------------

Before reading this tutorial, you should have seen :
 * :ref:`tuto01`


Structure
----------

Properties.cmake
~~~~~~~~~~~~~~~~~

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto02DataServiceBasic )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        servicesReg
        gui
        guiQt
        ioVTK # contains the reader and writer for VTK files (image and mesh).
        visuVTK # loads VTK rendering library (fwRenderVTK).
        visuVTKQt # contains the vtk Renderer window interactor manager using Qt.
        vtkSimpleNegato # contains a visualization service of medical image.
        fwlauncher
        appXml
    )

    bundleParam(appXml PARAM_LIST config PARAM_VALUES tutoDataServiceBasicConfig)

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``, the input file used to launch the application (see :ref:`profile.xml`).


plugin.xml
~~~~~~~~~~~

This file is located in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto02DataServiceBasic" version="@PROJECT_VERSION@">

        <!-- The bundles in requirements are automatically started when this
             application is launched. -->
        <requirement id="dataReg" />
        <requirement id="servicesReg" />
        <requirement id="visuVTKQt" />

        <extension implements="::fwServices::registry::AppConfig">
            <id>tutoDataServiceBasicConfig</id>
            <config>

                <!-- In tutoDataServiceBasic, the central data object is a ::fwData::Image. -->
                <object uid="imageData" type="::fwData::Image" />

                <!--
                    Description service of the GUI:
                    The ::gui::frame::SDefaultFrame service automatically positions the various
                    containers in the application main window.
                    Here, it declares a container for the 3D rendering service.
                -->
                <service uid="mainFrame" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>tutoDataServiceBasic</name>
                            <icon>Tuto02DataServiceBasic-0.1/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                    </gui>
                    <registry>
                        <!-- Associate the container for the rendering service. -->
                        <view sid="myRendering" />
                    </registry>
                </service>

                <!--
                    Reading service:
                    The <file> tag defines the path of the image to load. Here, it is a relative
                    path from the repository in which you launch the application.
                -->
                <service uid="myReaderPathFile" type="::ioVTK::SImageReader">
                    <inout key="data" uid="imageData" />
                    <file>../../data/patient1.vtk</file>
                </service>

                <!--
                    Visualization service of a 3D medical image:
                    This service renders the 3D image.
                -->
                <service uid="myRendering" type="::vtkSimpleNegato::SRenderer">
                    <in key="image" uid="imageData" />
                </service>

                <!--
                    Definition of the starting order of the different services:
                    The frame defines the 3D scene container, so it must be started first.
                    The services will be stopped symmetrically in the reverse order.
                -->
                <start uid="mainFrame" />
                <start uid="myReaderPathFile" />
                <start uid="myRendering" />

                <!--
                    Definition of the service to update:
                    The reading service loads the data on the update.
                    The render updates must be called after the reading of the image.
                -->
                <update uid="myReaderPathFile" />
                <update uid="myRendering" />

            </config>
        </extension>

    </plugin>



For this tutorial, we have only one object ``::fwData::Image`` and three services:
 * ``::gui::frame::SDefaultFrame``: frame service
 * ``::ioVTK::SImageReader``: reader for 3D VTK image
 * ``::vtkSimpleNegato::SRenderer``: renderer for 3D image

The following order of the configuration elements must be respected:
  #. ``<object>``
  #. ``<service>``
  #. ``<connect>`` (see :ref:`tuto04`)
  #. ``<start>``
  #. ``<update>``

.. note::
    To avoid the ``<start uid="myRendering" />``, the frame service can automatically start the rendering service: you
    just need to add the attribute ``start="yes"`` in the ``<view>`` tag.

Run
----

To run the application, you must call the following line in the install or build directory:

.. code::

    bin/tuto02dataservicebasic
