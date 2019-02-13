.. _tuto10:

*******************************************
[*Tuto10MatrixTransformInGS*] Generic scene
*******************************************

This tutorial explains how to use matrices and concatenation of matrices.

Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`generic_scene`
 * :ref:`tuto08`

Structure
=========


Properties.cmake
----------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto10MatrixTransformInGS )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        servicesReg
        gui
        guiQt
        ioData
        ioVTK
        uiIO
        uiVisuQt
        visuVTK
        visuVTKQt
        visuVTKAdaptor
        fwlauncher
        appXml
    )

    bundleParam(appXml PARAM_LIST config PARAM_VALUES Tuto10MatrixTransformInGS)

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.


plugin.xml
----------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <!--
            This tutorial explains how to perform matrix transformation using the generic scene.

            To use this application, you need to load a mesh.
    -->
    <plugin id="Tuto10MatrixTransformInGS" version="@PROJECT_VERSION@">
        <requirement id="dataReg" />
        <requirement id="servicesReg" />
        <requirement id="visuVTKQt" />

        <extension implements="::fwServices::registry::AppConfig">
            <id>Tuto10MatrixTransformInGS</id>
            <config>
                <!-- ***************************************** Begin Objects declaration ***************************************** -->
                <object uid="matrixA" type="::fwData::TransformationMatrix3D">
                    <matrix>
                        <![CDATA[
                            1 0 0 2
                            0 1 0 0
                            0 0 1 0
                            0 0 0 1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixB" type="::fwData::TransformationMatrix3D">
                    <matrix>
                        <![CDATA[
                            1 0 0 4
                            0 1 0 0
                            0 0 1 0
                            0 0 0 1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixC" type="::fwData::TransformationMatrix3D">
                    <matrix>
                        <![CDATA[
                            1 0 0 0
                            0 1 0 0
                            0 0 1 2
                            0 0 0 1
                        ]]>
                    </matrix>
                </object>

                <object uid="matrixD" type="::fwData::TransformationMatrix3D">
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

                <object uid="mesh" type="::fwData::Mesh" />
                <!-- ***************************************** End Objects declaration ******************************************* -->
                <!-- ***************************************** Begin layouts declaration ***************************************** -->
                <service uid="mainFrame" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>Tuto10MatrixTransformInGS</name>
                            <icon>Tuto10MatrixTransformInGS-0.1/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                        <menuBar/>
                    </gui>
                    <registry>
                        <menuBar sid="menuBar" start="yes" />
                        <view sid="mainView" start="yes" />
                    </registry>
                </service>

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
                            <menuItem name="Load Mesh" shortcut="Ctrl+O" />
                            <separator/>
                            <menuItem name="Quit" shortcut="Ctrl+Q" specialAction="QUIT" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="actionLoadMesh" start="yes" />
                        <menuItem sid="actionQuit" start="yes" />
                    </registry>
                </service>

                <service uid="actionLoadMesh" type="::gui::action::SStarter">
                    <start uid="readerPathFile" />
                </service>

                <service uid="actionQuit" type="::gui::action::SQuit" />
                <service uid="mainView" type="::gui::view::SDefaultView">
                    <gui>
                        <layout type="::fwGui::CardinalLayoutManager">
                            <view align="center" />
                            <view align="bottom" minHeight="40" position="0" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="genericScene" start="yes" />
                        <view sid="matrixEditor" start="yes" />
                    </registry>
                </service>

                <!-- ***************************************** End layouts declaration ***************************************** -->
                <!-- ***************************************** Begin services declarations    ************************************ -->
                <service uid="matrixEditor" type="::uiVisuQt::STransformEditor">
                    <inout key="matrix" uid="matrixE" />
                    <translation enabled="no" />
                    <rotation enabled="y" min="0" max="360" />
                </service>

                <service uid="readerPathFile" type="::uiIO::editor::SIOSelector">
                    <inout key="data" uid="mesh" />
                    <type mode="reader" />
                </service>

                <!-- ***************************************** Begin render scenes declarations    ***************************************** -->
                <!--
                    Generic scene:
                    This scene shows four times the same mesh but with a different matrix. It manages multiples transformation matrices.
                -->
                <!-- *************************** Begin generic scene *************************** -->

                <service uid="genericScene" type="::fwRenderVTK::SRender">
                    <scene renderMode="auto">
                        <picker id="picker" vtkclass="fwVtkCellPicker" />
                        <renderer id="default" background="#052833" />

                        <!-- Declare the vtk transform to use (it is optional) -->
                        <vtkObject id="mat1" class="vtkTransform" />
                        <vtkObject id="mat2" class="vtkTransform" />
                        <vtkObject id="mat3" class="vtkTransform" />
                        <vtkObject id="mat4" class="vtkTransform" />
                        <vtkObject id="mat5" class="vtkTransform" />

                        <!--
                            Declare the rotationMat1 as the concatenation of mat5, mat1 and mat4
                            rotationMat1 = mat5 x mat1 x mat4
                        -->
                        <vtkObject id="rotationMat1" class="vtkTransform">
                            <vtkTransform>
                                <concatenate>mat5</concatenate>
                                <concatenate>mat1</concatenate>
                                <concatenate>mat4</concatenate>
                            </vtkTransform>
                        </vtkObject>

                        <!-- rotationMat2 = inv(mat5) x mat2 x mat5 x mat5 x mat5 x mat4 x mat4 -->
                        <vtkObject id="rotationMat2" class="vtkTransform">
                            <vtkTransform>
                                <concatenate inverse="yes">mat5</concatenate>
                                <concatenate>mat2</concatenate>
                                <concatenate>mat5</concatenate>
                                <concatenate>mat5</concatenate>
                                <concatenate>mat5</concatenate>
                                <concatenate>mat4</concatenate>
                                <concatenate>mat4</concatenate>
                            </vtkTransform>
                        </vtkObject>

                        <!-- rotationMat3 = mat3 x mat5 x mat4 x mat4 x mat4 -->
                        <vtkObject id="rotationMat3" class="vtkTransform">
                            <vtkTransform>
                                <concatenate>mat3</concatenate>
                                <concatenate>mat5</concatenate>
                                <concatenate>mat4</concatenate>
                                <concatenate>mat4</concatenate>
                                <concatenate>mat4</concatenate>
                            </vtkTransform>
                        </vtkObject>

                        <adaptor uid="matrixAdaptorA" />
                        <adaptor uid="matrixAdaptorB" />
                        <adaptor uid="matrixAdaptorC" />
                        <adaptor uid="matrixAdaptorD" />
                        <adaptor uid="matrixAdaptorE" />
                        <adaptor uid="MeshAdaptor1" />
                        <adaptor uid="MeshAdaptor2" />
                        <adaptor uid="MeshAdaptor3" />
                        <adaptor uid="MeshAdaptor4" />
                    </scene>
                </service>

                <!--
                    Defines transform adaptors:
                    This adaptor works on a ::fwData::TransformationMatrix3D and manages a vtkTransform. When
                    the ::fwData::TransformationMatrix3D is modified, it updates the vtkTransform, and vice
                    versa.
                -->
                <service uid="matrixAdaptorA" type="::visuVTKAdaptor::STransform" autoConnect="yes">
                    <inout key="tm3d" uid="matrixA" />
                    <config renderer="default" picker="" transform="mat1" />
                </service>

                <service uid="matrixAdaptorB" type="::visuVTKAdaptor::STransform" autoConnect="yes">
                    <inout key="tm3d" uid="matrixB" />
                    <config renderer="default" picker="" transform="mat2" />
                </service>

                <service uid="matrixAdaptorC" type="::visuVTKAdaptor::STransform" autoConnect="yes">
                    <inout key="tm3d" uid="matrixC" />
                    <config renderer="default" picker="" transform="mat3" />
                </service>

                <service uid="matrixAdaptorD" type="::visuVTKAdaptor::STransform" autoConnect="yes">
                    <inout key="tm3d" uid="matrixD" />
                    <config renderer="default" picker="" transform="mat4" />
                </service>

                <service uid="matrixAdaptorE" type="::visuVTKAdaptor::STransform" autoConnect="yes">
                    <inout key="tm3d" uid="matrixE" />
                    <config renderer="default" picker="" transform="mat5" />
                </service>

                <service uid="MeshAdaptor1" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
                    <in key="mesh" uid="mesh" />
                    <config renderer="default" picker="" />
                </service>

                <service uid="MeshAdaptor2" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
                    <in key="mesh" uid="mesh" />
                    <config renderer="default" picker="" transform="rotationMat1" />
                </service>

                <service uid="MeshAdaptor3" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
                    <in key="mesh" uid="mesh" />
                    <config renderer="default" picker="" transform="rotationMat2" />
                </service>

                <service uid="MeshAdaptor4" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
                    <in key="mesh" uid="mesh" />
                    <config renderer="default" picker="" transform="rotationMat3" />
                </service>

                <!-- *************************** End generic scene *************************** -->

                <!-- ***************************************** End render scenes declaration ***************************************** -->
                <!-- ***************************************** End services declarations    ************************************************ -->
                <start uid="mainFrame" />

                <!-- VTK scene 'genericScene' -->
                <start uid="matrixAdaptorA" />
                <start uid="matrixAdaptorB" />
                <start uid="matrixAdaptorC" />
                <start uid="matrixAdaptorD" />
                <start uid="matrixAdaptorE" />
                <start uid="MeshAdaptor1" />
                <start uid="MeshAdaptor2" />
                <start uid="MeshAdaptor3" />
                <start uid="MeshAdaptor4" />

            </config>
        </extension>
    </plugin>

GUI
---

This tutorial uses a new service to manage matrices:

.. figure:: ../media/Tuto10MatrixTransformInGS.png
    :scale: 80
    :align: center

STransformEditor
~~~~~~~~~~~~~~~~

This editors regulates the position and rotation defined in a transformation matrix

.. code-block:: xml

    <service uid="..." type="::uiVisuQt::STransformEditor">
        <inout key="matrix" uid="..."/>
        <translation enabled="no" min="-300"/>
        <rotation enabled="yes" min="-180" max="180" />
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

Generic scene
-------------

This tutorial presents new services and objects that allow to apply a transformation matrix to a VTK adaptor.

VTKObject
~~~~~~~~~

.. code-block:: xml

    <vtkObject id="mat1" class="vtkTransform" />

This line declare a matrix called `mat1` that can be used in the scene has a vtkTransform.

.. code-block:: xml

    <vtkObject id="rotationMat1" class="vtkTransform">
        <vtkTransform>
            <concatenate>mat5</concatenate>
            <concatenate>mat1</concatenate>
            <concatenate>mat4</concatenate>
        </vtkTransform>
    </vtkObject>

Theses lines allow to concatenate matrix in a new object called `rotationMat1`,
this object can be used in the scene as a vtkTransform.

SMesh
~~~~~

This adaptor displays a `Mesh` in a generic scene

.. code-block:: xml

  <service uid="..." type="::visuVTKAdaptor::SMesh"  autoConnect="yes">
       <in key="mesh" uid="..." />
       <config renderer="default" transform="rotationMat1"/>
   </service>

transform
     Transform visually applied on the mesh, this matrix must be a vtkTransform.

TransformationMatrix3D
~~~~~~~~~~~~~~~~~~~~~~

This data represents a 4*4 3D transformation matrix.

.. code-block:: xml

    <object uid="" type="::fwData::TransformationMatrix3D">
        <matrix>
            <![CDATA[
                1 0 0 0
                0 1 0 0
                0 0 1 0
                0 0 0 1
            ]]>
        </matrix>
    </object>

STransform
~~~~~~~~~~

Adaptor binding a TransformationMatrix3D to a vtkTransform.

.. code-block:: xml

    <service uid="..." type="::visuVTKAdaptor::STransform" autoConnect="yes">
        <inout key="tm3d" uid="matrixA" />
        <config renderer="default" picker="" transform="mat1" />
    </service>

With this service, the `TransformationMatrix3D` is binded to the corresponding vtkTransform in the scene.

Run
===

To run the application, you must call the following line into the install or build directory:

.. code::

    bin/fwlauncher share/Tuto10MatrixTransformInGS-0.1/profile.xml
