.. _tuto08:

********************************************
[*Tuto08GenericScene*] Generic scene
********************************************

This tutorial explains how to use the generic scene.

.. figure:: ../media/tuto08GenericScene1.png
    :scale: 80
    :align: center

    Image and mesh

.. figure:: ../media/tuto08GenericScene2.png
    :scale: 80
    :align: center

    Mesh with texture


Prerequisites
===============

Before reading this tutorial, you should have seen :
 * :ref:`generic_scene`
 * :ref:`tuto06`

Structure
=============


Properties.cmake
------------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto08GenericScene )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( UNIQUE TRUE )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        servicesReg
        gui
        guiQt
        ioData # contains reader/writer for mesh (.trian) or matrix (.trf)
        ioVTK
        uiIO
        uiVisuQt # contains several editors for visualization
        uiImageQt # contains several editors on image
        visuVTKQt
        visuVTKAdaptor # contains adaptors for the generic scene
        ctrlSelection # contains services to manage object selection (and associated services)
        launcher
        appXml
    )

    bundleParam(appXml PARAM_LIST config PARAM_VALUES Tuto08GenericScene)

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.


plugin.xml
------------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <!--
        This tutorial shows a VTK scene containing a 3D image and a textured mesh.
        To use this application, you should open a 3D image, a mesh and/or a 2D texture image.
    -->
    <plugin id="Tuto08GenericScene" version="@PROJECT_VERSION@">
        <requirement id="dataReg" />
        <requirement id="servicesReg" />
        <requirement id="visuVTKQt" />
        <extension implements="::fwServices::registry::AppConfig">
            <id>Tuto08GenericScene</id>
            <config>
                <object uid="imageUID" type="::fwData::Image" />
                <object uid="meshUID" type="::fwData::Mesh" />
                <object uid="textureUID" type="::fwData::Image" />
                <service uid="ihm" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>Tuto08GenericScene</name>
                            <icon>Tuto08GenericScene-0.1/tuto.ico</icon>
                        </frame>
                        <menuBar/>
                    </gui>
                    <registry>
                        <menuBar sid="menuBar" start="yes" />
                        <view sid="mainView" start="yes" />
                    </registry>
                </service>

                <!-- Status bar used to display the progress bar for reading -->
                <service uid="progressBar" type="::gui::editor::SJobBar" />
                <service uid="menuBar" type="::gui::aspect::SDefaultMenuBar">
                    <gui>
                        <layout>
                            <menu name="File" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuFile" start="yes" />
                    </registry>
                </service>

                <service uid="menuFile" type="::gui::aspect::SDefaultMenu">
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
                        <menuItem sid="actionOpenImage" start="yes" />
                        <menuItem sid="actionOpenMesh" start="yes" />
                        <menuItem sid="actionOpenTexture" start="yes" />
                        <menuItem sid="actionQuit" start="yes" />
                    </registry>
                </service>

                <!-- Actions to call readers -->
                <service uid="actionOpenImage" type="::gui::action::SStarter">
                    <start uid="imageReader" />
                </service>

                <service uid="actionOpenMesh" type="::gui::action::SStarter">
                    <start uid="meshReader" />
                </service>

                <service uid="actionOpenTexture" type="::gui::action::SStarter">
                    <start uid="textureReader" />
                </service>

                <!-- Quit action -->
                <service uid="actionQuit" type="::gui::action::SQuit" />
                <!-- main view -->
                <service uid="mainView" type="::gui::view::SDefaultView">
                    <gui>
                        <layout type="::fwGui::CardinalLayoutManager">
                            <view align="center" />
                            <view align="bottom" minWidth="400" minHeight="30" resizable="no" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="genericScene" start="yes" />
                        <view sid="editorsView" start="yes" />
                    </registry>
                </service>

                <!-- View for editors to update image visualization -->
                <service uid="editorsView" type="::gui::view::SDefaultView">
                    <gui>
                        <layout type="::fwGui::LineLayoutManager">
                            <orientation value="horizontal" />
                            <view proportion="0" minWidth="30" />
                            <view proportion="0" minWidth="50" />
                            <view proportion="1" />
                            <view proportion="0" minWidth="30" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="sliceListEditor" start="yes" />
                        <view sid="showScanEditor" start="yes" />
                        <view sid="sliderIndexEditor" start="yes" />
                        <view sid="snapshotScene1Editor" start="yes" />
                    </registry>
                </service>

                <!--
                    Editor used for scene snapshot:
                    It allows to select the snapshot filename and emits a "snapped" signal with this path.
                -->
                <service uid="snapshotScene1Editor" type="::uiVisuQt::SnapshotEditor" />

                <!--
                    Generic scene:
                    This scene displays a 3D image and a textured mesh.
                -->
                <!-- *************************** Begin generic scene *************************** -->

                <service uid="genericScene" type="::fwRenderVTK::SRender" autoConnect="yes">
                    <scene>
                        <!-- Image picker -->
                        <picker id="myPicker" vtkclass="fwVtkCellPicker" />
                        <!-- Renderer -->
                        <renderer id="default" background="0.0" />

                        <!-- adaptor displayed in the scene -->
                        <adaptor uid="meshAdaptor" />
                        <adaptor uid="textureAdaptor" />
                        <adaptor uid="imageAdaptor" />
                        <adaptor uid="snapshotAdaptor" />
                    </scene>
                </service>

                <!-- Mesh adaptor -->
                <service uid="meshAdaptor" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
                    <in key="mesh" uid="meshUID" />
                    <config renderer="default" picker="" uvgen="sphere" />
                </service>

                <!-- Texture adaptor, used by mesh adaptor -->
                <service uid="textureAdaptor" type="::visuVTKAdaptor::STexture" autoConnect="yes">
                    <inout key="texture" uid="textureUID" />
                    <config renderer="default" picker="" filtering="linear" wrapping="repeat" />
                </service>

                <!-- 3D image negatoscope adaptor -->
                <service uid="imageAdaptor" type="::visuVTKAdaptor::SNegatoMPR" autoConnect="yes">
                    <inout key="image" uid="imageUID" />
                    <config renderer="default" picker="myPicker" mode="3d" slices="3" sliceIndex="axial" />
                </service>

                <!-- Snapshot adaptor: creates a snapshot of the scene. It has a slot "snap" that receives a path -->
                <service uid="snapshotAdaptor" type="::visuVTKAdaptor::SSnapshot">
                    <config renderer="default" />
                </service>

                <!-- *************************** End generic scene *************************** -->

                <!-- ************************************************
                                    Displayed objects
                    ************************************************* -->
                <!-- Image displayed in the scene -->
                <service uid="imageReader" type="::uiIO::editor::SIOSelector">
                    <inout key="data" uid="imageUID" />
                    <type mode="reader" />
                </service>

                <!--
                    Generic editor representing a menu button.
                    It sends a signal with the current selected item.
                -->
                <service uid="sliceListEditor" type="::guiQt::editor::SSelectionMenuButton">
                    <toolTip>Manage slice visibility</toolTip><!-- button tooltip -->
                    <selected>3</selected><!-- Default selection -->
                    <items>
                        <item text="One slice" value="1" /><!-- first item, if selected the emitted value is "1" -->
                        <item text="three slices" value="3" /><!-- second item, if selected the emitted value is "1" -->
                    </items>
                </service>

                <!--
                    Generic editor representing a simple button with an icon.
                    The button can be checkable. In this case it can have a second icon.
                    - It emits a signal "clicked" when it is clicked.
                    - It emits a signal "toggled" when it is checked/unchecked.

                    Here, this editor is used to show or hide the image. It is connected to the image adaptor.
                -->
                <service uid="showScanEditor" type="::guiQt::editor::SSignalButton">
                    <config>
                        <checkable>true</checkable>
                        <icon>media-0.1/icons/sliceHide.png</icon>
                        <icon2>media-0.1/icons/sliceShow.png</icon2>
                        <iconWidth>40</iconWidth>
                        <iconHeight>16</iconHeight>
                        <checked>true</checked>
                    </config>
                </service>

                <!-- Editor representing a slider to navigate into image slices -->
                <service uid="sliderIndexEditor" type="::uiImageQt::SliceIndexPositionEditor" autoConnect="yes">
                    <inout key="image" uid="imageUID" />
                    <sliceIndex>axial</sliceIndex>
                </service>

                <!-- texture reader -->
                <service uid="textureReader" type="::uiIO::editor::SIOSelector">
                    <inout key="data" uid="textureUID" />
                    <type mode="reader" />
                </service>

                <!-- Mesh reader -->
                <service uid="meshReader" type="::uiIO::editor::SIOSelector">
                    <inout key="data" uid="meshUID" />
                    <type mode="reader" />
                </service>

                <!-- Connects readers to status bar service -->
                <connect>
                    <signal>meshReader/jobCreated</signal>
                    <slot>progressBar/showJob</slot>
                </connect>

                <connect>
                    <signal>imageReader/jobCreated</signal>
                    <slot>progressBar/showJob</slot>
                </connect>

                <connect>
                    <signal>textureReader/jobCreated</signal>
                    <slot>progressBar/showJob</slot>
                </connect>

                <!--
                    Connects showScanEditor signal "toggled" to sliceListEditor slot "setEnable", this signal and slot
                    contains a boolean, so the sliceListEditor can be disabled when the image is not displayed.
                -->
                <connect>
                    <signal>showScanEditor/toggled</signal>
                    <slot>sliceListEditor/setEnabled</slot>
                </connect>

                <!--
                    Connection for snapshot:
                    connect the editor signal "snapped" to the adaptor slot "snap"
                -->
                <connect>
                    <signal>snapshotScene1Editor/snapped</signal>
                    <slot>snapshotAdaptor/snap</slot>
                </connect>

                <!--
                    Connection for 3D image slice:
                    Connect the button (showScanEditor) signal "toggled" to the image adaptor (SNegatoMPR)
                    slot "showSlice", this signals/slots contains a boolean.
                    The image slices will be shown or hidden when the button is checked/unchecked.
                -->
                <connect>
                    <signal>showScanEditor/toggled</signal>
                    <slot>imageAdaptor/showSlice</slot>
                </connect>

                <!--
                    Connection for 3D image slice:
                    Connect the menu button (sliceListEditor) signal "selected" to the image adaptor
                    (SNegatoMPR) slot "updateSliceMode", this signals/slots contains an integer.
                    This integer defines the number of slice to show (0, 1 or 3).
                -->
                <connect>
                    <signal>sliceListEditor/selected</signal>
                    <slot>imageAdaptor/updateSliceMode</slot>
                </connect>

                <!--
                    Connection for texture:
                    The texture will be applied on the mesh when the mesh adaptor is started.
                -->
                <connect>
                    <signal>meshAdaptor/textureApplied</signal>
                    <slot>textureAdaptor/applyTexture</slot>
                </connect>

                <start uid="ihm" />
                <start uid="progressBar" />

                <!-- genericScene adaptors-->
                <start uid="meshAdaptor" />
                <start uid="textureAdaptor" />
                <start uid="imageAdaptor" />
                <start uid="snapshotAdaptor" />
            </config>
        </extension>
    </plugin>


GUI
------

This tutorials use multiple editors to manage the image rendering:

- show/hide image slices
- navigate between the image slices
- snapshot

.. figure:: ../media/tuto08GenericSceneGUI.png
    :scale: 80
    :align: center

The two editors (``SSelectionMenuButton`` and ``SSignalButton``) are generic, so we need to configure their behaviour in
the xml file.

The editor aspect is defined in the service configuration. They emit signals that must be manually connected to the
scene adaptor.

SSelectionMenuButton
~~~~~~~~~~~~~~~~~~~~~~~

This editor displays a menu when the user click on the button. Then the user can select one item.

.. figure:: ../media/SSelectionMenuButton.png
    :align: center

.. code-block:: xml

    <service uid="selectionMenuButton" impl="::guiQt::editor::SSelectionMenuButton">
        <text>...</text>
        <toolTip>...</toolTip>
        <items>
            <item text="One" value="1" />
            <item text="Two" value="2" />
            <item text="Six" value="6" />
        </items>
        <selected>2</selected>
    </service>

text (optional, default ">")
    Text displayed on the button

toolTip (optional)
    Button tool tip

items
    List of the menu items

item
    One item

    text
        The text displayed in the menu
    value
        The value emitted when the item is selected

selected
    The value of the item selected by default


When the user selects an item, a signal is emitted: the signal is ``selected(int selection)``. It sends the value of
the selected item.

In our case, we want to change the number of image slices displayed in the scene. So, we need to connect this signal to
the image adaptor slot ``updateSliceMode(int nbSlice)``.

.. code-block:: xml

    <connect>
        <signal>selectionMenuButton/selected</signal>
        <slot>imageAdaptor/updateSliceMode</slot>
    </connect>


SSignalButton
~~~~~~~~~~~~~~~

This editor shows a simple button.

.. code-block:: xml

    <service uid="signalButton" impl="::guiQt::editor::SSignalButton" >
        <config>
            <checkable>true|false</checkable>
            <text>...</text>
            <icon>...</icon>
            <text2>...</text2>
            <icon2>...</icon2>
            <checked>true|false</checked>
            <iconWidth>...</iconWidth>
            <iconHeight>...</iconHeight>
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
        <signal>signalButton/toggled</signal>
        <slot>imageAdaptor/showSlice</slot>
    </connect>

Run
=========

To run the application, you must call the following line into the install or build directory:

.. code::

    bin/fwlauncher share/Tuto08GenericScene-0.1/profile.xml
