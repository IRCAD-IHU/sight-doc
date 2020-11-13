.. _TutorialsTuto09MatrixTransformInGS:

*******************************************
[*Tuto09MatrixTransformInGS*] Generic scene
*******************************************

This tutorial explains how to use matrices and concatenation of matrices.

.. figure:: ../media/tuto09MatrixTransformInGS.png
    :scale: 25
    :align: center

=============
Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`TutorialsTuto07GenericScene`
 * :ref:`SADGenericScene`

=========
Structure
=========

----------------
Properties.cmake
----------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto09MatrixTransformInGS )
    set( VERSION 0.2 )
    set( TYPE APP )
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

        # Reader
        ioData
        ioVTK
        ioAtoms

        # Services
        uiIO
        uiVisuQt
        maths

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
            Tuto09MatrixTransformInGS_AppCfg
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
        This tutorial explains how to perform matrix transformation using the generic scene.

        To use this application, you need to load a mesh.
    -->
    <plugin id="Tuto09MatrixTransformInGS" version="@PROJECT_VERSION@" >

        <requirement id="preferences" />
        <requirement id="visuOgre" />
        <requirement id="material" />
        <requirement id="guiQt" />
        <requirement id="servicesReg" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto09MatrixTransformInGS_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <object uid="matrixA" type="::fwData::TransformationMatrix3D" >
                    <matrix>
                        <![CDATA[
                            1 0 0 2
                            0 1 0 0
                            0 0 1 0
                            0 0 0 1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixB" type="::fwData::TransformationMatrix3D" >
                    <matrix>
                        <![CDATA[
                            1 0 0 4
                            0 1 0 0
                            0 0 1 0
                            0 0 0 1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixC" type="::fwData::TransformationMatrix3D" >
                    <matrix>
                        <![CDATA[
                            1 0 0 0
                            0 1 0 0
                            0 0 1 2
                            0 0 0 1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixD" type="::fwData::TransformationMatrix3D" >
                    <matrix>
                        <![CDATA[
                            0.75 0        0        0
                            0        0.75 0        0
                            0        0        0.75 0
                            0        0        0    1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixE" type="::fwData::TransformationMatrix3D" />

                <object uid="rotation1" type="::fwData::TransformationMatrix3D" />

                <object uid="rotation2" type="::fwData::TransformationMatrix3D" />

                <object uid="rotation3" type="::fwData::TransformationMatrix3D" />

                <object uid="mesh" type="::fwData::Mesh" />

                <!-- ******************************* UI declaration *********************************** -->

                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto09MatrixTransformInGS</name>
                            <icon>Tuto09MatrixTransformInGS-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                        <menuBar/>
                    </gui>
                    <registry>
                        <menuBar sid="menuBarView" start="yes" />
                        <view sid="mainView" start="yes" />
                    </registry>
                </service>

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
                            <menuItem name="Open mesh" shortcut="Ctrl+O" />
                            <separator/>
                            <menuItem name="Quit" shortcut="Ctrl+Q" specialAction="QUIT" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="openMeshAct" start="yes" />
                        <menuItem sid="quitAct" start="yes" />
                    </registry>
                </service>

                <service uid="mainView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="vertical" />
                            <view proportion="1" />
                            <view proportion="0" minHeight="30" resizable="no" backgroundColor="#3E4453" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="genericSceneSrv" start="yes" />
                        <view sid="matrixEditorSrv" start="yes" />
                    </registry>
                </service>

                <!-- *************************** Begin generic scene *************************** -->

                <!--
                    Generic scene:
                    This scene shows four times the same mesh but with a different matrix. It manages multiples transformation matrices.
                -->
                <service uid="genericSceneSrv" type="::fwRenderOgre::SRender" >
                    <scene>
                        <background topColor="#36393E" bottomColor="#36393E" />

                        <layer id="default" order="1" />
                        <adaptor uid="trackballInteractorAdp" />
                        <adaptor uid="mesh0Adp" />
                        <adaptor uid="transform1Adp" />
                        <adaptor uid="mesh1Adp" />
                        <adaptor uid="transform2Adp" />
                        <adaptor uid="mesh2Adp" />
                        <adaptor uid="transform3Adp" />
                        <adaptor uid="mesh3Adp" />
                    </scene>
                </service>

                <service uid="trackballInteractorAdp" type="::visuOgreAdaptor::STrackballCamera" >
                    <config layer="default" priority="0" />
                </service>

                <service uid="mesh0Adp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
                    <inout key="mesh" uid="mesh" />
                    <config layer="default" />
                </service>

                <service uid="transform1Adp" type="::visuOgreAdaptor::STransform" autoConnect="yes" >
                    <inout key="transform" uid="rotation1" />
                    <config layer="default" transform="rotationTransform1" />
                </service>

                <service uid="mesh1Adp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
                    <inout key="mesh" uid="mesh" />
                    <config layer="default" transform="rotationTransform1" />
                </service>

                <service uid="transform2Adp" type="::visuOgreAdaptor::STransform" autoConnect="yes" >
                    <inout key="transform" uid="rotation2" />
                    <config layer="default" transform="rotationTransform2" />
                </service>

                <service uid="mesh2Adp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
                    <inout key="mesh" uid="mesh" />
                    <config layer="default" transform="rotationTransform2" />
                </service>

                <service uid="transform3Adp" type="::visuOgreAdaptor::STransform" autoConnect="yes" >
                    <inout key="transform" uid="rotation3" />
                    <config layer="default" transform="rotationTransform3" />
                </service>

                <service uid="mesh3Adp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
                    <inout key="mesh" uid="mesh" />
                    <config layer="default" transform="rotationTransform3" />
                </service>

                <!-- ******************************* Actions ****************************************** -->

                <service uid="openMeshAct" type="::gui::action::SStarter" >
                    <start uid="meshReaderSrv" />
                </service>

                <service uid="quitAct" type="::gui::action::SQuit" />

                <!-- ******************************* Services ***************************************** -->

                <service uid="matrixEditorSrv" type="::uiVisuQt::STransformEditor" >
                    <inout key="matrix" uid="matrixE" />
                    <translation enabled="no" />
                    <rotation enabled="y" min="0" max="360" />
                </service>

                <service uid="meshReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="mesh" />
                    <type mode="reader" />
                </service>

                <service uid="rotation1Srv" type="::maths::SConcatenateMatrices" >
                    <in group="matrix" >
                        <key uid="matrixE" autoConnect="yes" />
                        <key uid="matrixA" />
                        <key uid="matrixD" />
                    </in>
                    <inout key="output" uid="rotation1" />
                </service>

                <service uid="rotation2Srv" type="::maths::SConcatenateMatrices" >
                    <in group="matrix" >
                        <key uid="matrixB" />
                        <key uid="matrixE" autoConnect="yes" />
                        <key uid="matrixE" />
                        <key uid="matrixE" />
                        <key uid="matrixD" />
                        <key uid="matrixD" />
                    </in>
                    <inout key="output" uid="rotation2" />
                </service>

                <service uid="rotation3Srv" type="::maths::SConcatenateMatrices" >
                    <in group="matrix" >
                        <key uid="matrixC" />
                        <key uid="matrixE" autoConnect="yes" />
                        <key uid="matrixD" />
                        <key uid="matrixD" />
                        <key uid="matrixD" />
                    </in>
                    <inout key="output" uid="rotation3" />
                </service>

                <!-- ******************************* Start services ***************************************** -->

                <start uid="mainFrame" />
                <start uid="rotation1Srv" />
                <start uid="rotation2Srv" />
                <start uid="rotation3Srv" />
                <start uid="trackballInteractorAdp" />
                <start uid="mesh0Adp" />
                <start uid="transform1Adp" />
                <start uid="mesh1Adp" />
                <start uid="transform2Adp" />
                <start uid="mesh2Adp" />
                <start uid="transform3Adp" />
                <start uid="mesh3Adp" />

                <update uid="rotation1Srv" />
                <update uid="rotation2Srv" />
                <update uid="rotation3Srv" />

            </config>
        </extension>
    </plugin>

===
GUI
===

This tutorial uses new services to manage matrices:

----------------
STransformEditor
----------------

This editors regulates the position and rotation defined in a transformation matrix

.. code-block:: xml

    <service uid="matrixEditorSrv" type="::uiVisuQt::STransformEditor" >
        <inout key="matrix" uid="matrixE" />
        <translation enabled="no" />
        <rotation enabled="y" min="0" max="360" />
    </service>

translation
    Used to updates the translation of the matrix

rotation
    Used to updates the rotation of the matrix

enable (optional, default "yes")
    Enables/disables rotation/translation edition. Can be 'yes', 'no' or a combination of [xyz]

min (optional, default "translation=-300, rotation=-180")
    Sets the minimum value for translation/rotation

max (optional, default "translation=300, rotation=180")
    Sets the maximum value for translation/rotation

When the user move the slider, the matrix given in the `inout` section is updated.

=============
Generic scene
=============

This tutorial presents new services and objects that allow to apply a transformation matrix to a VTK adaptor.

----------------------
TransformationMatrix3D
----------------------

This data represents a 4*4 3D transformation matrix.

.. code-block:: xml

    <object uid="matrixA" type="::fwData::TransformationMatrix3D">
        <matrix>
            <![CDATA[
                1 0 0 0
                0 1 0 0
                0 0 1 0
                0 0 0 1
            ]]>
        </matrix>
    </object>

----------
STransform
----------

Adaptor binding a TransformationMatrix3D to a generic scene.

.. code-block:: xml

    <service uid="transformAdp" type="::visuOgreAdaptor::STransform" autoConnect="yes" >
        <inout key="transform" uid="matrixA" />
        <config layer="default" transform="rotationTransform" />
    </service>

With this service, the `TransformationMatrix3D` is binded to the corresponding given name (`rotationTransform`).

-----
SMesh
-----

This adaptor displays a `Mesh` in a generic scene

.. code-block:: xml

    <service uid="meshAdp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
        <inout key="mesh" uid="mesh" />
        <config layer="default" transform="rotationTransform" />
    </service>

transform
    Transform visually applied on the mesh, the name must match the name given
    in the transform adaptor (`rotationTransform`).

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto09matrixtransformings

   .. group-tab:: Windows

        .. code::

            bin/tuto09matrixtransformings.bat
