.. _tuto05:

**********************************************
[*Tuto05Mesher*] Create a mesh from an image
**********************************************

The fifth tutorial explains how to use several objects in an application. 
This application provides an action to creates a mesh from an image.

.. figure:: ../media/tuto05Mesher.png
    :scale: 80
    :align: center


Prerequisites
===============

Before to read this tutorial, you should have seen :
 * :ref:`tuto04`


Structure
=============

Properties.cmake
------------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto05Mesher )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        servicesReg
        gui
        guiQt
        io
        ioVTK
        visuVTKQt
        uiIO
        vtkSimpleNegato
        vtkSimpleMesh
        opVTKMesh # provides services to generate a mesh from an image.
        launcher
        appXml2
    )

    bundleParam(appXml2 PARAM_LIST config PARAM_VALUES MesherConfig)


.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.


plugin.xml
------------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto05Mesher" version="@DASH_VERSION@">

        <requirement id="dataReg" />
        <requirement id="servicesReg" />
        <requirement id="visuVTKQt" />

        <extension implements="::fwServices::registry::AppConfig2">
            <id>MesherConfig</id>
            <config>

                <!-- Mesh object associated to the uid 'myMesh' -->
                <object uid="myMesh" type="::fwData::Mesh" />

                <!-- Image object associated to the key 'myImage' -->
                <object uid="myImage" type="::fwData::Image" />

                <!-- Frame & View -->

                <service uid="myFrame" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>Mesher</name>
                            <icon>@BUNDLE_PREFIX@/Tuto05Mesher_0-1/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                        <menuBar />
                    </gui>
                    <registry>
                        <menuBar sid="myMenuBar" start="yes" />
                        <view sid="myDefaultView" start="yes" />
                    </registry>
                </service>

                <!--
                    Default view service:
                    The type '::fwGui::LineLayoutManager' represents a layout where the view are aligned
                    horizontally or vertically (set orientation value 'horizontal' or 'vertical').
                    It is possible to add a 'proportion' attribute for the <view> to defined the proportion
                    used by the view compared to the others.
                -->
                <service uid="myDefaultView" type="::gui::view::SDefaultView">
                    <gui>
                        <layout type="::fwGui::LineLayoutManager">
                            <orientation value="horizontal" />
                            <view caption="Image view" />
                            <view caption="Mesh view" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="RenderingImage" start="yes" />
                        <view sid="RenderingMesh" start="yes" />
                    </registry>
                </service>

                <!-- Menu Bar, Menus & Actions -->

                <service uid="myMenuBar" type="::gui::aspect::SDefaultMenuBar">
                    <gui>
                        <layout>
                            <menu name="File" />
                            <menu name="Mesher" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuFile" start="yes" />
                        <menu sid="menuMesher" start="yes" />
                    </registry>
                </service>

                <service uid="menuFile" type="::gui::aspect::SDefaultMenu">
                    <gui>
                        <layout>
                            <menuItem name="Open image file" shortcut="Ctrl+O" />
                            <menuItem name="Save image" />
                            <separator />
                            <menuItem name="Open mesh file" shortcut="Ctrl+M" />
                            <menuItem name="Save mesh" />
                            <separator />
                            <menuItem name="Quit" specialAction="QUIT" shortcut="Ctrl+Q" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="actionOpenImageFile" start="yes" />
                        <menuItem sid="actionSaveImageFile" start="yes" />
                        <menuItem sid="actionOpenMeshFile" start="yes" />
                        <menuItem sid="actionSaveMeshFile" start="yes" />
                        <menuItem sid="actionQuit" start="yes" />
                    </registry>
                </service>

                <service uid="menuMesher" type="::gui::aspect::SDefaultMenu">
                    <gui>
                        <layout>
                            <menuItem name="Compute Mesh (VTK)" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="actionCreateVTKMesh" start="yes" />
                    </registry>
                </service>

                <service uid="actionQuit" type="::gui::action::SQuit" />

                <service uid="actionOpenImageFile" type="::gui::action::SStarter">
                    <start uid="readerPathImageFile" />
                </service>

                <service uid="actionSaveImageFile" type="::gui::action::SStarter">
                    <start uid="writerImageFile" />
                </service>

                <service uid="actionOpenMeshFile" type="::gui::action::SStarter">
                    <start uid="readerPathMeshFile" />
                </service>

                <service uid="actionSaveMeshFile" type="::gui::action::SStarter">
                    <start uid="writerMeshFile" />
                </service>

                <service uid="actionCreateVTKMesh" type="::opVTKMesh::action::SMeshCreation">
                    <in key="image" uid="myImage" />
                    <inout key="mesh" uid="myMesh" />
                    <percentReduction value="0" />
                </service>

                <!--
                    Services associated to the Image data :
                    Visualization, reading and writing service creation.
                -->
                <service uid="RenderingImage" type="::vtkSimpleNegato::SRenderer" autoConnect="yes" >
                    <in key="image" uid="myImage" />
                </service>

                <service uid="readerPathImageFile" type="::uiIO::editor::SIOSelector">
                    <inout key="target" uid="myImage" />
                    <type mode="reader" />
                </service>

                <service uid="writerImageFile" type="::uiIO::editor::SIOSelector">
                    <in key="target" uid="myImage" />
                    <type mode="writer" />
                </service>

                <!--
                    Services associated to the Mesh data :
                    Visualization, reading and writing service creation.
                -->
                <service uid="RenderingMesh" type="::vtkSimpleMesh::SRenderer" autoConnect="yes" >
                    <in key="mesh" uid="myMesh" />
                </service>

                <service uid="readerPathMeshFile" type="::uiIO::editor::SIOSelector">
                    <inout key="target" uid="myMesh" />
                    <type mode="reader" />
                </service>

                <service uid="writerMeshFile" type="::uiIO::editor::SIOSelector">
                    <in key="target" uid="myMesh" />
                    <type mode="writer" />
                </service>

                <start uid="myFrame" />

            </config>
        </extension>
    </plugin>



Run
=========

To run the application, you must call the following line into the install or build directory:

.. code::

    bin/fwlauncher Bundles/Tuto05Mesher_0-1/profile.xml
