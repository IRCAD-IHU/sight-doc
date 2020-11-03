.. _TutorialsTuto07GenericScene:

************************************
[*Tuto07GenericScene*] Generic scene
************************************

This tutorial explains how to use the generic scene.

.. figure:: ../media/tuto07GenericScene1.png
    :scale: 25
    :align: center

    Image

.. figure:: ../media/tuto07GenericScene2.png
    :scale: 25
    :align: center

    Mesh

.. figure:: ../media/tuto07GenericScene3.png
    :scale: 25
    :align: center

    Mesh with texture

=============
Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`TutorialsTuto06Filter`
 * :ref:`SADGenericScene`

=========
Structure
=========

----------------
Properties.cmake
----------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto07GenericScene )
    set( VERSION 0.2 )
    set( TYPE APP )
    set( UNIQUE TRUE )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        fwlauncher              # Needed to build the launcher
        appXml                  # XML configurations

        preferences             # Start the module, load file location or window preferences
        guiQt                   # Start the module, load qt implementation of gui
        visuOgre                # Start the module, allow to use fwRenderOgre
        material                # Start the module, load Ogre's materials
        visuOgreQt              # Enable Ogre to render things in Qt window

        # Objects declaration
        fwData
        servicesReg             # fwService

        # UI declaration/Actions
        gui
        style
        flatIcon

        # Reader
        ioData
        ioVTK
        ioAtoms

        # Services
        uiIO
        uiImageQt

        # Generic Scene
        visuOgreAdaptor
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
            Tuto07GenericScene_AppCfg
    ) # Main application's configuration to launch

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.

----------
plugin.xml
----------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <!--
        This tutorial shows a VTK scene containing a 3D image and a textured mesh.
        To use this application, you should open a 3D image, a mesh and/or a 2D texture image.
    -->
    <plugin id="Tuto07GenericScene" version="@PROJECT_VERSION@" >

        <requirement id="preferences" />
        <requirement id="visuOgre" />
        <requirement id="material" />
        <requirement id="guiQt" />
        <requirement id="servicesReg" />
        <requirement id="visuOgreQt" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto07GenericScene_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <object uid="image" type="::fwData::Image" />
                <object uid="mesh" type="::fwData::Mesh" />
                <object uid="texture" type="::fwData::Image" />
                <object uid="snapshot" type="::fwData::Image" />

                <!-- ******************************* UI declaration *********************************** -->

                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto07GenericScene</name>
                            <icon>Tuto07GenericScene-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="720" height="480" />
                        </frame>
                        <menuBar/>
                    </gui>
                    <registry>
                        <menuBar sid="menuBarView" start="yes" />
                        <view sid="containerView" start="yes" />
                    </registry>
                </service>

                <!-- Status bar used to display the progress bar for reading -->
                <service uid="progressBarView" type="::gui::editor::SJobBar" />

                <service uid="menuBarView" type="::gui::aspect::SDefaultMenuBar" >
                    <gui>
                        <layout>
                            <menu name="File" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuFileView" start="yes" />
                    </registry>
                </service>

                <service uid="menuFileView" type="::gui::aspect::SDefaultMenu" >
                    <gui>
                        <layout>
                            <menuItem name="Open image" shortcut="Ctrl+I" />
                            <menuItem name="Open mesh" shortcut="Ctrl+M" />
                            <menuItem name="Open texture" shortcut="Ctrl+T" />
                            <separator/>
                            <menuItem name="Quit" specialAction="QUIT" shortcut="Ctrl+Q" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="openImageAct" start="yes" />
                        <menuItem sid="openMeshAct" start="yes" />
                        <menuItem sid="openTextureAct" start="yes" />
                        <menuItem sid="quitAct" start="yes" />
                    </registry>
                </service>

                <!-- main view -->
                <service uid="containerView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="vertical" />
                            <view proportion="1" />
                            <view proportion="0" minHeight="30" resizable="no" backgroundColor="#3E4453" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="genericSceneSrv" start="yes" />
                        <view sid="editorsView" start="yes" />
                    </registry>
                </service>

                <!-- View for editors to update image visualization -->
                <service uid="editorsView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="horizontal" />
                            <view proportion="0" minWidth="50" />
                            <view proportion="1" />
                            <view proportion="0" minWidth="30" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="showNegatoSrv" start="yes" />
                        <view sid="sliderIndexEditorSrv" start="yes" />
                        <view sid="snapshotSrv" start="yes" />
                    </registry>
                </service>

                <!-- *************************** Begin generic scene *************************** -->

                <!-- This scene display a 3D image and a textured mesh -->
                <service uid="genericSceneSrv" type="::fwRenderOgre::SRender" >
                    <scene>
                        <background topColor="#36393E" bottomColor="#36393E" />

                        <layer id="default" order="1" />
                        <adaptor uid="trackballInteractorAdp" />
                        <adaptor uid="textureAdp" />
                        <adaptor uid="meshAdp" />
                        <adaptor uid="negatoAdp" />
                        <adaptor uid="snapshotAdp" />
                    </scene>
                </service>

                <service uid="trackballInteractorAdp" type="::visuOgreAdaptor::STrackballCamera" >
                    <config layer="default" priority="0" />
                </service>

                <!-- Texture adaptor, used by mesh adaptor -->
                <service uid="textureAdp" type="::visuOgreAdaptor::STexture" autoConnect="yes" >
                    <in key="image" uid="texture" />
                    <config layer="default" textureName="ogreTexture" />
                </service>

                <!-- Mesh adaptor -->
                <service uid="meshAdp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
                    <inout key="mesh" uid="mesh" />
                    <config layer="default" textureName="ogreTexture" />
                </service>

                <!-- 3D image negatoscope adaptor -->
                <service uid="negatoAdp" type="::visuOgreAdaptor::SNegato3D" autoConnect="yes" >
                    <inout key="image" uid="image" />
                    <config layer="default" sliceIndex="axial" interactive="true" />
                </service>

                <service uid="snapshotAdp" type="::visuOgreAdaptor::SFragmentsInfo" >
                    <inout key="image" uid="snapshot" />
                    <config layer="default" flip="true" />
                </service>

                <!-- ******************************* Actions ****************************************** -->

                <!-- Actions to call readers -->
                <service uid="openImageAct" type="::gui::action::SStarter" >
                    <start uid="imageReaderSrv" />
                </service>

                <service uid="openMeshAct" type="::gui::action::SStarter" >
                    <start uid="meshReaderSrv" />
                </service>

                <service uid="openTextureAct" type="::gui::action::SStarter" >
                    <start uid="textureReaderSrv" />
                </service>

                <!-- Quit action -->
                <service uid="quitAct" type="::gui::action::SQuit" />

                <!-- ******************************* Services ***************************************** -->

                <!-- Image displayed in the scene -->
                <service uid="imageReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="image" />
                    <type mode="reader" />
                </service>

                <!-- Mesh reader -->
                <service uid="meshReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="mesh" />
                    <type mode="reader" />
                </service>

                <!-- texture reader -->
                <service uid="textureReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="texture" />
                    <type mode="reader" />
                </service>

                <!--
                    Generic editor representing a simple button with an icon.
                    The button can be checkable. In this case it can have a second icon.
                    - It emits a signal "clicked" when it is clicked.
                    - It emits a signal "toggled" when it is checked/unchecked.

                    Here, this editor is used to show or hide the image. It is connected to the image adaptor.
                -->
                <service uid="showNegatoSrv" type="::guiQt::editor::SSignalButton" >
                    <config>
                        <checkable>true</checkable>
                        <icon>flatIcon-0.1/icons/RedCross.svg</icon>
                        <icon2>flatIcon-0.1/icons/Layers.svg</icon2>
                        <iconWidth>40</iconWidth>
                        <iconHeight>16</iconHeight>
                        <checked>true</checked>
                    </config>
                </service>

                <!-- Editor representing a slider to navigate into image slices -->
                <service uid="sliderIndexEditorSrv" type="::uiImageQt::SliceIndexPositionEditor" autoConnect="yes" >
                    <inout key="image" uid="image" />
                    <sliceIndex>axial</sliceIndex>
                </service>

                <service uid="snapshotSrv" type="::guiQt::editor::SSignalButton" >
                    <config>
                       <checkable>false</checkable>
                       <icon>flatIcon-0.1/icons/YellowPhoto.svg</icon>
                    </config>
                </service>

                <!-- Write the snapshot image -->
                <service uid="imageWriterSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="snapshot" />
                    <type mode="writer" />
                    <selection mode="exclude" />
                    <addSelection service="::ioAtoms::SWriter" />
                </service>

                <!-- ******************************* Connections ***************************************** -->

                <!-- Connects readers to status bar service -->
                <connect>
                    <signal>meshReaderSrv/jobCreated</signal>
                    <signal>imageReaderSrv/jobCreated</signal>
                    <signal>textureReaderSrv/jobCreated</signal>
                    <slot>progressBarView/showJob</slot>
                </connect>

                <connect>
                    <signal>snapshotSrv/clicked</signal>
                    <slot>imageWriterSrv/update</slot>
                </connect>

                <!--
                    Connection for 3D image slice:
                    Connect the button (showNegatoSrv) signal "toggled" to the negato adaptor (SNegato3D)
                    slot "updateVisibility", this signals/slots contains a boolean.
                    The image slices will be show or hide when the button is checked/unchecked.
                -->
                <connect>
                    <signal>showNegatoSrv/toggled</signal>
                    <slot>negatoAdp/updateVisibility</slot>
                </connect>

                <!-- ******************************* Start services ***************************************** -->

                <start uid="mainFrame" />
                <start uid="progressBarView" />
                <start uid="imageWriterSrv" />
                <start uid="trackballInteractorAdp" />
                <start uid="textureAdp" />
                <start uid="meshAdp" />
                <start uid="negatoAdp" />
                <start uid="snapshotAdp" />

            </config>
        </extension>
    </plugin>

===
GUI
===

This tutorials use multiple editors to manage the image rendering:

- show/hide image slices
- navigate between the image slices
- snapshot

The two editors (``SSignalButton``) are generic, so we need to configure their behaviour in
the xml file.

The editor aspect is defined in the service configuration. They emit signals that must be manually connected to the
scene adaptor.

-------------
SSignalButton
-------------

This editor shows a simple button.

.. code-block:: xml

    <service uid="showNegatoSrv" type="::guiQt::editor::SSignalButton" >
        <config>
            <checkable>true</checkable>
            <icon>flatIcon-0.1/icons/RedCross.svg</icon>
            <icon2>flatIcon-0.1/icons/Layers.svg</icon2>
            <iconWidth>40</iconWidth>
            <iconHeight>16</iconHeight>
            <checked>true</checked>
        </config>
    </service>

text (optional)
    Text displayed on the button

icon (optional)
    Icon displayed on the button

checkable (optional, default: false)
    If true, the button is checkable

text2 (optional)
    Text displayed if the button is checked

icon2 (optional)
    Icon displayed if the button is checked

checked (optional, default: false)
    If true, the button is checked at start

iconWidth (optional)
    Icon width

iconHeight (optional)
    Icon height

This editor provides two signals:

clicked()
    Emitted when the user click on the button.

toggled(bool checked)
    Emitted when the button is checked or unchecked.

In our case, we want to show (or hide) the image slices when the button is checked (or unckecked). So, we need to
connect the ``toogled`` signal to the image adaptor slot ``showSlice(bool show)``.

.. code-block:: xml

    <connect>
        <signal>showNegatoSrv/toggled</signal>
        <slot>negatoAdp/updateVisibility</slot>
    </connect>

------------------------
SliceIndexPositionEditor
------------------------

This editor allows to change the slice index of an image.

.. code-block:: xml

    <service uid="sliderIndexEditorSrv" type="::uiImageQt::SliceIndexPositionEditor" autoConnect="yes" >
        <inout key="image" uid="image" />
        <sliceIndex>axial</sliceIndex>
    </service>

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto07genericscene

   .. group-tab:: Windows

        .. code::

            bin/tuto07genericscene.bat
