.. _TutorialsTuto05Mesher:

********************************************
[*Tuto05Mesher*] Create a mesh from an image
********************************************

The fifth tutorial explains how to use several objects in an application.
This application provides an action to create a mesh from an image mask.

.. figure:: ../media/tuto05Mesher.png
    :scale: 25
    :align: center

=============
Prerequisites
=============

Before reading this tutorial, you should have seen:
 * :ref:`TutorialsTuto04SignalSlot`

=========
Structure
=========

----------------
Properties.cmake
----------------

This file describes the project information and requirements:

.. code-block:: cmake

    set( NAME Tuto05Mesher )
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
        style

        # Reader
        ioVTK

        # Services
        uiIO
        visuBasic               # Contains a visualization service of mesh.
        opVTKMesh               # Provides services to generate a mesh from an image.
    )

    moduleParam(guiQt
        PARAM_LIST
            resource
            stylesheet
        PARAM_VALUES
            style-0.1/flatdark.rcc
            style-0.1/flatdark.qss
    ) # Allow dark theme via guiQt

    moduleParam(
            appXml
        PARAM_LIST
            config
        PARAM_VALUES
            Tuto05Mesher_AppCfg
    ) # Main application's configuration to launch

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.

----------
plugin.xml
----------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto05Mesher" version="@PROJECT_VERSION@" >

        <requirement id="servicesReg" />
        <requirement id="guiQt" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto05Mesher_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <!-- Mesh object associated to the uid 'myMesh' -->
                <object uid="myMesh" type="::fwData::Mesh" />

                <!-- Image object associated to the key 'myImage' -->
                <object uid="myImage" type="::fwData::Image" />

                <!-- ******************************* UI declaration *********************************** -->

                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto05Mesher</name>
                            <icon>Tuto05Mesher-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                        <menuBar />
                    </gui>
                    <registry>
                        <menuBar sid="menuBarView" start="yes" />
                        <view sid="containerView" start="yes" />
                    </registry>
                </service>

                <service uid="menuBarView" type="::gui::aspect::SDefaultMenuBar" >
                    <gui>
                        <layout>
                            <menu name="File" />
                            <menu name="Mesher" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuFileView" start="yes" />
                        <menu sid="menuMesherView" start="yes" />
                    </registry>
                </service>

                <service uid="menuFileView" type="::gui::aspect::SDefaultMenu" >
                    <gui>
                        <layout>
                            <menuItem name="Open image" shortcut="Ctrl+O" />
                            <menuItem name="Save image" />
                            <separator />
                            <menuItem name="Open mesh" shortcut="Ctrl+M" />
                            <menuItem name="Save mesh" />
                            <separator />
                            <menuItem name="Quit" specialAction="QUIT" shortcut="Ctrl+Q" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="openImageAct" start="yes" />
                        <menuItem sid="saveImageAct" start="yes" />
                        <menuItem sid="openMeshAct" start="yes" />
                        <menuItem sid="saveMeshAct" start="yes" />
                        <menuItem sid="quitAct" start="yes" />
                    </registry>
                </service>

                <service uid="menuMesherView" type="::gui::aspect::SDefaultMenu" >
                    <gui>
                        <layout>
                            <menuItem name="Compute Mesh (VTK)" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="createMeshAct" start="yes" />
                    </registry>
                </service>

                <!--
                    Default view service:
                    The type '::fwGui::LineLayoutManager' represents a layout where the view are aligned
                    horizontally or vertically (set orientation value 'horizontal' or 'vertical').
                    It is possible to add a 'proportion' attribute for the <view> to defined the proportion
                    used by the view compared to the others.
                -->
                <service uid="containerView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="horizontal" />
                            <view/>
                            <view/>
                        </layout>
                    </gui>
                    <registry>
                        <view sid="imageSrv" start="yes" />
                        <view sid="meshSrv" start="yes" />
                    </registry>
                </service>

                <!-- ******************************* Actions ****************************************** -->

                <service uid="quitAct" type="::gui::action::SQuit" />

                <service uid="openImageAct" type="::gui::action::SStarter" >
                    <start uid="imageReaderSrv" />
                </service>

                <service uid="saveImageAct" type="::gui::action::SStarter" >
                    <start uid="imageWriterSrv" />
                </service>

                <service uid="openMeshAct" type="::gui::action::SStarter" >
                    <start uid="meshReaderSrv" />
                </service>

                <service uid="saveMeshAct" type="::gui::action::SStarter" >
                    <start uid="meshWriterSrv" />
                </service>

                <service uid="createMeshAct" type="::opVTKMesh::action::SMeshCreation" >
                    <in key="image" uid="myImage" />
                    <inout key="mesh" uid="myMesh" />
                    <percentReduction value="0" />
                </service>

                <!-- ******************************* Services ***************************************** -->

                <!-- Add a shortcut in the application (key "v") -->
                <service uid="shortcutSrv" type="::guiQt::SSignalShortcut" >
                    <config shortcut="v" sid="containerView" />
                </service>

                <!--
                    Services associated to the Image data :
                    Visualization, reading and writing service creation.
                -->
                <service uid="imageSrv" type="::visuBasic::SImage" >
                    <in key="image" uid="myImage" autoConnect="yes" />
                </service>

                <service uid="imageReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="myImage" />
                    <type mode="reader" />
                </service>

                <service uid="imageWriterSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="myImage" />
                    <type mode="writer" />
                </service>

                <!--
                    Services associated to the Mesh data :
                    Visualization, reading and writing service creation.
                -->
                <service uid="meshSrv" type="::visuBasic::SMesh" >
                    <in key="mesh" uid="myMesh" autoConnect="yes" />
                </service>

                <service uid="meshReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="myMesh" />
                    <type mode="reader" />
                </service>

                <service uid="meshWriterSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="myMesh" />
                    <type mode="writer" />
                </service>

                <!-- ******************************* Connections ***************************************** -->

                <!-- Connect the shortcut "v" to the update slot of 'createMeshAct'-->
                <connect>
                    <signal>shortcutSrv/activated</signal>
                    <slot>createMeshAct/update</slot>
                </connect>

                <!-- ******************************* Start services ***************************************** -->

                <start uid="mainFrame" />
                <start uid="shortcutSrv" />

            </config>
        </extension>
    </plugin>

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto05mesher

   .. group-tab:: Windows

        .. code::

            bin/tuto05mesher.bat
