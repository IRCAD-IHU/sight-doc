.. _TutorialsTuto17SimpleAR:

*********************************************************
[*Tuto17SimpleAR*] Augmented-reality with *ArUco* markers
*********************************************************

This tutorial shows a basic sample of augmented reality.
This exhibits:

- how to detect an *ArUco* marker,
- how to compute the pose of a marker,
- how to make a basic augmented-reality render (we superimpose a plane onto the *ArUco* marker),
- how to undistort a video according to the intrinsic parameters of the camera,
- how to distort a 3D render according to the intrinsic parameters of the camera,
- how to synchronize a video process pipeline with the video playback efficiently, using ``SSignalGate``.

To use this application, you must open a calibration and a video. Samples are provided in the module folder
of the appplication, ``share/sight/TutoSimpleAR-0.4`` on Linux and ``share\TutoSimpleAR-0.4`` on Windows:

- ``calibration.xml``
- ``aruco_tag.m4v``

.. raw:: html

       <video width="700" controls>
          <source src="https://owncloud.ircad.fr/index.php/s/AWNwGSeaPOJeXzE/download" >
          Your browser does not support the video tag.
       </video>

=============
Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`TutorialsTuto07GenericScene`

=========
Structure
=========

----------------
Properties.cmake
----------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto17SimpleAR )
    set( VERSION 0.5 )
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
        arData
        servicesReg             # fwService

        # UI declaration/Actions
        gui
        style
        flatIcon

        # Reader
        ioData
        ioVTK
        ioAtoms

        # Grabber
        videoQt

        # Services
        ioCalibration
        uiTools
        ctrlCamp
        videoTools
        trackerAruco
        registrationCV
        videoCalibration
        maths
        ctrlCom

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
            Tuto17SimpleAR_AppCfg
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
        This tutorial shows a basic sample of augmented reality.
        This exhibits:
          - how to detect an ArUco marker,
          - how to compute the pose of a marker,
          - how to make a basic augmented-reality render (we superimpose a cube onto the ArUco marker)
          - how to undistort a video according to the camera intrinsic parameters
          - how to distort a 3D render according to the camera intrinsic parameters
          - how to synchronize efficiently a video process pipeline with the video playback using SSignalGate

        To use this application, you must open a calibration and a video. Samples are provided in the module folder
        of the application, `share/sight/Tuto17SimpleAR-0.4` on Linux/MacOs and `share\Tuto17SimpleAR-0.4` on Windows:
          - calibration.xml
          - aruco_tag.m4v
    -->
    <plugin id="Tuto17SimpleAR" version="@PROJECT_VERSION@" >

        <requirement id="preferences" />
        <requirement id="visuOgre" />
        <requirement id="material" />
        <requirement id="guiQt" />
        <requirement id="servicesReg" />
        <requirement id="preferences" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto17SimpleAR_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <!-- Series of camera calibration -->
                <object uid="cameraSeries" type="::arData::CameraSeries" />
                <!-- Camera calibration, deferred because it is extracted from the series -->
                <object uid="camera" type="::arData::Camera" src="deferred" />
                <!-- Frame timeline used to buffer the output of the video grabber -->
                <object uid="frameTL" type="::arData::FrameTL" />
                <!-- Video frame that is used for the current iteration -->
                <object uid="sourceFrame" type="::fwData::Image" />
                <!-- Map containing all detected markers in the current frame -->
                <object uid="markerMap" type="::arData::MarkerMap" />
                <!-- Marker matrix in the camera world -->
                <object uid="markerToCamera" type="::fwData::TransformationMatrix3D" />
                <!-- Camera matrix in the marker world -->
                <object uid="cameraToMarker" type="::fwData::TransformationMatrix3D" />
                <!-- Cube superimposed on the video at the marker location -->
                <object uid="cubeMesh" type="::fwData::Mesh" />
                <object uid="undistortMap" type="::fwData::Image" />
                <object uid="distortMap" type="::fwData::Image" />

                <!-- ******************************* UI declaration *********************************** -->

                <!-- declaration of the views, menu and toolbar -->
                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto17SimpleAR</name>
                            <icon>Tuto17SimpleAR-@PROJECT_VERSION@/app.ico</icon>
                        </frame>
                        <toolBar/>
                    </gui>
                    <registry>
                        <toolBar sid="toolbarView" start="yes" />
                        <view sid="cameraView" start="yes" />
                    </registry>
                </service>

                <service uid="toolbarView" type="::gui::aspect::SDefaultToolBar" >
                    <gui>
                        <layout>
                            <menuItem name="Load Calibration" icon="flatIcon-0.1/icons/YellowCamera.svg" />
                            <editor/>
                            <separator/>
                            <menuItem name="Start" icon="flatIcon-0.1/icons/GreenStart.svg" shortcut="Space" />
                            <menuItem name="Pause" icon="flatIcon-0.1/icons/OrangePause.svg" shortcut="Space" />
                            <menuItem name="Play" icon="flatIcon-0.1/icons/GreenStart.svg" shortcut="Space" />
                            <menuItem name="Stop" icon="flatIcon-0.1/icons/RedStop.svg" />
                            <menuItem name="Loop" icon="flatIcon-0.1/icons/OrangeLoop.svg" style="check" />
                            <separator />
                            <menuItem name="Enable video image undistortion" icon="flatIcon-0.1/icons/YellowUndistortion.svg" />
                            <menuItem name="Disable video image undistortion" icon="flatIcon-0.1/icons/YellowDistortion.svg" />
                            <menuItem name="Enable 3D rendering distortion" icon="flatIcon-0.1/icons/YellowDistortion.svg" />
                            <menuItem name="Disable 3D rendering distortion" icon="flatIcon-0.1/icons/YellowUndistortion.svg" />
                            <separator/>
                            <menuItem name="Show Mesh on tag" icon="flatIcon-0.1/icons/Mask.svg" style="check" />
                            <separator/>
                            <menuItem name="Settings" icon="flatIcon-0.1/icons/BlueParameters.svg" style="check" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="loadCalibAct" start="yes" />
                        <editor sid="videoSelectorSrv" />
                        <menuItem sid="startVideoAct" start="yes" />
                        <menuItem sid="pauseVideoAct" start="yes" />
                        <menuItem sid="resumeVideoAct" start="yes" />
                        <menuItem sid="stopVideoAct" start="yes" />
                        <menuItem sid="loopVideoAct" start="yes" />
                        <menuItem sid="startUndistortAct" start="yes" />
                        <menuItem sid="stopUndistortAct" start="yes" />
                        <menuItem sid="startDistortAct" start="yes" />
                        <menuItem sid="stopDistortAct" start="yes" />
                        <menuItem sid="showMeshAct" start="yes" />
                        <menuItem sid="showParametersAct" start="yes" />
                    </registry>
                </service>

                <service uid="cameraView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="horizontal" />
                            <view proportion="1" />
                            <view proportion="0" backgroundColor="#36393E" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="videoView" start="yes" />
                        <view sid="parametersView" start="yes" />
                    </registry>
                </service>

                <service uid="videoView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="vertical" />
                            <view proportion="3" />
                            <view proportion="0" backgroundColor="#36393E" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="genericSceneSrv" />
                        <view sid="errorLabelSrv" start="yes" />
                    </registry>
                </service>

                <service uid="parametersView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::LineLayoutManager" >
                            <orientation value="vertical" />
                            <view proportion="3" />
                            <view proportion="2" />
                            <spacer/>
                        </layout>
                    </gui>
                    <registry>
                        <view sid="arucoParamametersSrv" start="yes" />
                        <view sid="reprojectionParamametersSrv" start="yes" />
                    </registry>
                </service>

                <!-- ************************************* Action ************************************ -->

                <!-- declaration of actions/slot callers -->
                <service uid="showParametersAct" type="::gui::action::SModifyLayout" >
                    <config>
                        <show_or_hide sid="parametersView" />
                    </config>
                </service>

                <service uid="loadCalibAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>calibrationReaderSrv/update</slot>
                    </slots>
                </service>

                <!-- Start the frame grabber -->
                <service uid="startVideoAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>videoGrabberSrv/startCamera</slot>
                    </slots>
                    <state executable="false" />
                </service>

                <!-- Pause the frame grabber -->
                <service uid="pauseVideoAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>videoGrabberSrv/pauseCamera</slot>
                        <slot>resumeVideoAct/show</slot>
                        <slot>pauseVideoAct/hide</slot>
                    </slots>
                    <state visible="false" />
                </service>

                <!-- Resume the frame grabber -->
                <service uid="resumeVideoAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>videoGrabberSrv/pauseCamera</slot>
                        <slot>resumeVideoAct/hide</slot>
                        <slot>pauseVideoAct/show</slot>
                    </slots>
                    <state visible="false" />
                </service>

                <!-- Stop the frame grabber -->
                <service uid="stopVideoAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>videoGrabberSrv/stopCamera</slot>
                        <slot>startVideoAct/show</slot>
                        <slot>resumeVideoAct/hide</slot>
                        <slot>pauseVideoAct/hide</slot>
                        <slot>stopVideoAct/setInexecutable</slot>
                        <slot>loopVideoAct/setInexecutable</slot>
                        <slot>loopVideoAct/deactivate</slot>
                    </slots>
                    <state executable="false" />
                </service>

                <!-- Loop the frame grabber -->
                <service uid="loopVideoAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>videoGrabberSrv/loopVideo</slot>
                    </slots>
                    <state executable="false" />
                </service>

                <service uid="startUndistortAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>undistortAdp/show</slot>
                        <slot>startDistortAct/setInexecutable</slot>
                        <slot>startUndistortAct/hide</slot>
                        <slot>stopUndistortAct/show</slot>
                    </slots>
                </service>

                <service uid="stopUndistortAct" type="::gui::action::SSlotCaller" >
                    <state visible="false" />
                    <slots>
                        <slot>undistortAdp/hide</slot>
                        <slot>startDistortAct/setExecutable</slot>
                        <slot>startUndistortAct/show</slot>
                        <slot>stopUndistortAct/hide</slot>
                    </slots>
                </service>

                <service uid="startDistortAct" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>distortAdp/show</slot>
                        <slot>startUndistortAct/setInexecutable</slot>
                        <slot>startDistortAct/hide</slot>
                        <slot>stopDistortAct/show</slot>
                    </slots>
                </service>

                <service uid="stopDistortAct" type="::gui::action::SSlotCaller" >
                    <state visible="false" />
                    <slots>
                        <slot>distortAdp/hide</slot>
                        <slot>startUndistortAct/setExecutable</slot>
                        <slot>startDistortAct/show</slot>
                        <slot>stopDistortAct/hide</slot>
                    </slots>
                </service>

                <service uid="showMeshAct" type="::gui::action::SBooleanSlotCaller" >
                    <slots>
                        <slot>cubeAdp/updateVisibility</slot>
                    </slots>
                    <state active="true" />
                </service>

                <!-- ******************************* Begin Generic Scene ******************************* -->

                <!-- Scene in which the video and the 3D will be rendered -->
                <!-- In this tutorial, we move the camera and the marker mesh is fixed -->
                <service uid="genericSceneSrv" type="::fwRenderOgre::SRender" >
                    <!-- It is essential to use the 'sync' mode when doing AR -->
                    <!-- In this mode, the renderer will wait for a signal to trigger the rendering -->
                    <scene renderMode="sync" >
                        <background topColor="#36393E" bottomColor="#36393E" />

                        <layer id="video" order="1" compositors="Remap" />
                        <adaptor uid="videoAdp" />
                        <adaptor uid="undistortAdp" />

                        <layer id="default" order="3" compositors="Remap" />
                        <adaptor uid="axisAdp" />
                        <adaptor uid="cameraAdp" />
                        <adaptor uid="cubeAdp" />
                        <adaptor uid="distortAdp" />
                    </scene>
                </service>

                <service uid="videoAdp" type="::visuOgreAdaptor::SVideo" autoConnect="yes" >
                    <in key="image" uid="sourceFrame" />
                    <config layer="video" />
                </service>

                <service uid="undistortAdp" type="::visuOgreAdaptor::SCompositorParameter" autoConnect="yes" >
                    <inout key="parameter" uid="undistortMap" />
                    <config layer="video" compositorName="Remap" parameter="u_map" shaderType="fragment" visible="false" />
                </service>

                <service uid="axisAdp" type="::visuOgreAdaptor::SAxis" >
                    <config layer="default" length="30" origin="true" label="false" />
                </service>

                <!-- Camera for the 3D layer -->
                <service uid="cameraAdp" type="::visuOgreAdaptor::SCamera" autoConnect="yes" >
                    <inout key="transform" uid="cameraToMarker" />
                    <in key="calibration" uid="camera" />
                    <config layer="default" />
                </service>

                <!-- Cube displayed on top of the marker plane -->
                <service uid="cubeAdp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
                    <inout key="mesh" uid="cubeMesh" />
                    <config layer="default" autoresetcamera="no" />
                </service>

                <service uid="distortAdp" type="::visuOgreAdaptor::SCompositorParameter" autoConnect="yes" >
                    <inout key="parameter" uid="distortMap" />
                    <config layer="default" compositorName="Remap" parameter="u_map" shaderType="fragment" visible="false" />
                </service>

                <!-- ************************************* Services ************************************ -->

                <service uid="loadMeshSrv" type="::ioVTK::SMeshReader" >
                    <inout key="data" uid="cubeMesh" />
                    <resource>Tuto17SimpleAR-@PROJECT_VERSION@/cube_60.vtk</resource>
                </service>

                <!-- hide axis adaptor until a marker is found -->
                <service uid="hideAxisSrv" type="::gui::action::SBooleanSlotCaller" >
                    <slots>
                        <slot>axisAdp/updateVisibility</slot>
                    </slots>
                </service>

                <!-- Calibration reader (here OpenCV's XML/YAML) -->
                <service uid="calibrationReaderSrv" type="::ioCalibration::SOpenCVReader" >
                    <inout key="data" uid="cameraSeries" />
                </service>

                <!-- extract the first ::arData::Camera from the ::arData::CameraSeries -->
                <service uid="extractCameraSrv" type="::ctrlCamp::SExtractObj" >
                    <inout key="source" uid="cameraSeries" >
                        <extract from="@cameras.0" /> <!-- Camp path of the first camera in cameraSeries -->
                    </inout>
                    <out group="target" >
                        <key uid="camera" /> <!-- destination -->
                    </out>
                </service>

                <!-- GUI to handle aruco tracking parameters -->
                <service uid="arucoParamametersSrv" type="::guiQt::editor::SParameters" >
                    <parameters>
                        <!-- show marker or not -->
                        <param type="bool" name="Show Marker" key="debugMode" defaultValue="true" />
                        <!--  do corner refinement or not. -->
                        <param type="bool" name="Corner refinement." key="corner" defaultValue="true" />
                        <!-- minimum window size for adaptive thresholding before finding contours -->
                        <param type="int" name="adpt. Threshold win size min" key="adaptiveThreshWinSizeMin" defaultValue="3" min="3" max="100" />
                        <!-- maximum window size for adaptive thresholding before finding contours -->
                        <param type="int" name="adpt. Threshold win size max" key="adaptiveThreshWinSizeMax" defaultValue="23" min="4" max="100" />
                        <!-- increments from adaptiveThreshWinSizeMin to adaptiveThreshWinSizeMax during the thresholding -->
                        <param type="int" name="adpt. Threshold win size step" key="adaptiveThreshWinSizeStep" defaultValue="10" min="1" max="100" />
                        <!-- constant for adaptive thresholding before finding contours -->
                        <param type="double" name="adpt. threshold constant" key="adaptiveThreshConstant" defaultValue="7." min="0." max="30." />
                        <!-- determine minimum perimeter for marker contour to be detected.
                            This is defined as a rate respect to the maximum dimension of the input image -->
                        <param type="double" name="Min. Marker Perimeter Rate" key="minMarkerPerimeterRate" defaultValue="0.03" min="0.01" max="1.0" />
                        <!-- determine maximum perimeter for marker contour to be detected.
                            This is defined as a rate respect to the maximum dimension of the input image -->
                        <param type="double" name="Max. Marker Perimeter Rate" key="maxMarkerPerimeterRate" defaultValue="4.0" min="1." max="10." />
                        <!-- minimum accuracy during the polygonal approximation process to determine which contours are squares -->
                        <param type="double" name="Polygonal Approx. Accuracy Rate" key="polygonalApproxAccuracyRate" defaultValue="0.03" min="0.01" max="1." />
                        <!-- minimum distance between corners for detected markers relative to its perimeter -->
                        <param type="double" name="Min. Corner Distance Rate" key="minCornerDistanceRate" defaultValue="0.01" min="0." max="1." />
                        <!-- minimum distance of any corner to the image border for detected markers (in pixels) -->
                        <param type="int" name="Min. Distance to Border" key="minDistanceToBorder" defaultValue="1" min="0" max="10" />
                        <!-- minimum mean distance beetween two marker corners to be considered similar,
                        so that the smaller one is removed. The rate is relative to the smaller perimeter of the two markers -->
                        <param type="double" name="Min. Marker Distance Rate" key="minMarkerDistanceRate" defaultValue="0.01" min="0." max="1." />
                        <!-- window size for the corner refinement process (in pixels) -->
                        <param type="int" name="Corner Refinement Win. Size" key="cornerRefinementWinSize" defaultValue="5" min="1" max="100" />
                        <!-- maximum number of iterations for stop criteria of the corner refinement process -->
                        <param type="int" name="Corner Refinement Max Iterations" key="cornerRefinementMaxIterations" defaultValue="30" min="1" max="10" />
                        <!-- minimum error for the stop cristeria of the corner refinement process -->
                        <param type="double" name="Corner Refinement Min. Accuracy" key="cornerRefinementMinAccuracy" defaultValue="0.1" min="0.01" max="10." />
                        <!-- number of bits of the marker border, i.e. marker border width -->
                        <param type="int" name="Marker Border Bits" key="markerBorderBits" defaultValue="1" min="1" max="100" />
                        <!-- number of bits (per dimension) for each cell of the marker when removing the perspective -->
                        <param type="int" name="Perspective Remove Pixel per Cell" key="perspectiveRemovePixelPerCell" defaultValue="8" min="1" max="32" />
                        <!-- width of the margin of pixels on each cell not considered for the determination of the cell bit.
                            Represents the rate respect to the total size of the cell,
                            i.e. perpectiveRemovePixelPerCel -->
                        <param type="double" name="Perspective Remove Ignored Margin Per Cell" key="perspectiveRemoveIgnoredMarginPerCell" defaultValue="0.13" min="0." max="1." />
                        <!-- maximum number of accepted erroneous bits in the border (i.e. number of allowed white bits in the border).
                            Represented as a rate respect to the total number of bits per marker -->
                        <param type="double" name="Max. Erroneous Bits In Border Rate" key="maxErroneousBitsInBorderRate" defaultValue="0.35" min="0." max="1." />
                        <!-- minimun standard deviation in pixels values during the decodification step to apply Otsu thresholding
                            (otherwise, all the bits are set to 0 or 1 depending on mean higher than 128 or not) -->
                        <param type="double" name="Min. Otsu Std. Dev." key="minOtsuStdDev" defaultValue="5.0" min="0." max="100." />
                        <!-- error correction rate respect to the maximun error correction capability for each dictionary -->
                        <param type="double" name="Error Correction Rate" key="errorCorrectionRate" defaultValue="0.6" min="0." max="1." />
                    </parameters>
                    <config sendAtStart="false" />
                </service>

                <service uid="reprojectionParamametersSrv" type="::guiQt::editor::SParameters" >
                    <parameters>
                        <param type="bool" name="Show reprojection" key="display" defaultValue="true" />
                        <param type="color" name="Circle color" key="color" defaultValue="#ffffff" />
                    </parameters>
                    <config sendAtStart="false" />
                </service>

                <!-- Gui Service to display a value in a QLabel -->
                <service uid="errorLabelSrv" type="::uiTools::editor::STextStatus" >
                    <label>Reprojection error (RMSE)</label>
                    <color>#D25252</color>
                </service>

                <!-- GUI to select camera (device, file, or stream) -->
                <service uid="videoSelectorSrv" type="::videoQt::editor::SCamera" >
                    <inout key="camera" uid="camera" />
                    <videoSupport>yes</videoSupport>
                </service>

                <!-- Grab image from camera device and fill a frame timeline -->
                <service uid="videoGrabberSrv" type="::videoQt::SFrameGrabber" >
                    <in key="camera" uid="camera" />
                    <inout key="frameTL" uid="frameTL" />
                </service>

                <!-- Consumes a frame in the timeline, picks the latest one to be processed -->
                <!-- It is overkill in this sample, but mandatory when we use multiple grabbers to synchronize them. -->
                <service uid="frameUpdaterSrv" type="::videoTools::SFrameMatrixSynchronizer" >
                    <in group="frameTL" >
                        <key uid="frameTL" autoConnect="yes" />
                    </in>
                    <inout group="image" >
                        <key uid="sourceFrame" />
                    </inout>
                    <tolerance>100</tolerance>
                    <framerate>0</framerate>
                </service>

                <!-- Aruco tracker service -->
                <service uid="trackerSrv" type="::trackerAruco::SArucoTracker" worker="tracking" >
                    <in key="camera" uid="camera" />
                    <inout key="frame" uid="sourceFrame" autoConnect="yes" />
                    <inout group="markerMap" >
                        <key uid="markerMap" /> <!-- timeline of detected tag(s) -->
                    </inout>
                    <track>
                        <!-- list of tag's id -->
                        <marker id="104" />
                    </track>
                    <debugMarkers>yes</debugMarkers>
                </service>

                <!-- Computes the pose of the camera with tag(s) detected by aruco -->
                <service uid="registrationSrv" type="::registrationCV::SPoseFrom2d" worker="tracking" >
                    <in group="markerMap" autoConnect="yes" >
                        <key uid="markerMap" />
                    </in>
                    <in group="camera" >
                        <key uid="camera" />
                    </in>
                    <inout group="matrix" >
                        <key uid="markerToCamera" id="104" />
                    </inout>
                    <patternWidth>60</patternWidth>
                </service>

                <!-- Computes the reprojection error -->
                <service uid="errorSrv" type="::videoCalibration::SReprojectionError" worker="error" >
                    <in group="matrix" autoConnect="yes" >
                        <key uid="markerToCamera" id="104" />
                    </in>
                    <in key="markerMap" uid="markerMap" />
                    <in key="camera" uid="camera" />
                    <inout key="frame" uid="sourceFrame" />
                    <patternWidth>60</patternWidth>
                </service>

                <!-- Multiply matrices (here only used to inverse "markerToCamera") -->
                <service uid="matrixReverserSrv" type="::maths::SConcatenateMatrices" >
                    <in group="matrix" >
                        <key uid="markerToCamera" autoConnect="yes" inverse="true" />
                    </in>
                    <inout key="output" uid="cameraToMarker" />
                </service>

                <service uid="undistorterSrv" type="::videoCalibration::SDistortion" >
                    <in key="camera" uid="camera" autoConnect="yes" />
                    <inout key="map" uid="undistortMap" />
                    <mode>undistort</mode>
                </service>

                <service uid="distorterSrv" type="::videoCalibration::SDistortion" >
                    <in key="camera" uid="camera" autoConnect="yes" />
                    <inout key="map" uid="distortMap" />
                    <mode>distort</mode>
                </service>

                <!-- Wait for the undistortion and the matrix inversion to be finished -->
                <service uid="syncGenericSceneSrv" type="::ctrlCom::SSignalGate" >
                    <signal>sourceFrame/bufferModified</signal>
                    <signal>cameraToMarker/modified</signal>
                </service>

                <!-- ******************************* Connections ***************************************** -->

                <connect>
                    <signal>videoSelectorSrv/configuredFile</signal>
                    <signal>videoSelectorSrv/configuredStream</signal>
                    <signal>videoSelectorSrv/configuredDevice</signal>
                    <slot>startVideoAct/update</slot>
                </connect>

                <connect>
                    <signal>videoGrabberSrv/cameraStarted</signal>
                    <slot>pauseVideoAct/show</slot>
                    <slot>startVideoAct/hide</slot>
                    <slot>stopVideoAct/setExecutable</slot>
                    <slot>loopVideoAct/setExecutable</slot>
                </connect>

                <connect>
                    <signal>camera/idModified</signal>
                    <slot>videoGrabberSrv/stopCamera</slot>
                </connect>

                <connect>
                    <signal>camera/modified</signal>
                    <slot>startVideoAct/setExecutable</slot>
                    <slot>stopVideoAct/update</slot>
                </connect>

                <!-- signal/slot connection -->
                <connect>
                    <signal>reprojectionParamametersSrv/colorChanged</signal>
                    <slot>errorSrv/setColorParameter</slot>
                </connect>

                <connect>
                    <signal>reprojectionParamametersSrv/boolChanged</signal>
                    <slot>errorSrv/setBoolParameter</slot>
                </connect>

                <connect>
                    <signal>errorSrv/errorComputed</signal>
                    <slot>errorLabelSrv/setDoubleParameter</slot>
                </connect>

                <connect>
                    <signal>arucoParamametersSrv/boolChanged</signal>
                    <slot>trackerSrv/setBoolParameter</slot>
                </connect>

                <connect>
                    <signal>arucoParamametersSrv/intChanged</signal>
                    <slot>trackerSrv/setIntParameter</slot>
                </connect>

                <connect>
                    <signal>arucoParamametersSrv/doubleChanged</signal>
                    <slot>trackerSrv/setDoubleParameter</slot>
                </connect>

                <connect>
                    <signal>cameraSeries/modified</signal>
                    <slot>extractCameraSrv/update</slot>
                    <slot>errorSrv/update</slot>
                </connect>

                <connect>
                    <signal>trackerSrv/markerDetected</signal>
                    <slot>axisAdp/updateVisibility</slot>
                </connect>

                <!-- When the undistortion and the matrix inversion are done, trigger the rendering -->
                <!-- then process a new frame -->
                <connect>
                    <signal>syncGenericSceneSrv/allReceived</signal>
                    <slot>genericSceneSrv/requestRender</slot>
                    <slot>frameUpdaterSrv/synchronize</slot>
                </connect>

                <!-- ******************************* Start services ***************************************** -->

                <start uid="mainFrame" />
                <start uid="videoGrabberSrv" />
                <start uid="frameUpdaterSrv" />
                <start uid="registrationSrv" />
                <start uid="trackerSrv" />
                <start uid="calibrationReaderSrv" />
                <start uid="videoSelectorSrv" />
                <start uid="extractCameraSrv" />
                <start uid="matrixReverserSrv" />
                <start uid="errorSrv" />
                <start uid="hideAxisSrv" />
                <start uid="undistorterSrv" />
                <start uid="distorterSrv" />
                <start uid="syncGenericSceneSrv" />
                <start uid="loadMeshSrv" />
                <start uid="genericSceneSrv" />
                <start uid="videoAdp" />
                <start uid="undistortAdp" />
                <start uid="axisAdp" />
                <start uid="cameraAdp" />
                <start uid="cubeAdp" />
                <start uid="distortAdp" />

                <!-- ******************************* Update services ***************************************** -->

                <!-- At launch, enable the synchronization with the non-offscreen rendering -->
                <update uid="showParametersAct" />
                <update uid="hideAxisSrv" />
                <update uid="loadMeshSrv" />
                <update uid="arucoParamametersSrv" />
                <update uid="reprojectionParamametersSrv" />

            </config>
        </extension>
    </plugin>

========
Tracking
========

Tag detection is done with the ``::trackerAruco::SArucoTracker`` service, which takes a video frame
as input, and fills in a map of identified *ArUco* markers. You only have to specify which marker identifier you
want to retrieve, here we choose the tag **101** because it is the one seen in the video sample.

.. code-block:: xml

    <service uid="trackerSrv" type="::trackerAruco::SArucoTracker" worker="tracking" >
        <in key="camera" uid="camera" />
        <inout key="frame" uid="sourceFrame" autoConnect="yes" />
        <inout group="markerMap" >
            <key uid="markerMap" /> <!-- timeline of detected tag(s) -->
        </inout>
        <track>
            <!-- list of tag's id -->
            <marker id="104" />
        </track>
        <debugMarkers>yes</debugMarkers>
    </service>

The map of markers is a ``::arData::MarkerMap``, which stores, for each tag identifier, a list of 2D coordinates
corresponding to the shape of the marker. In the *ArUco* case, the markers are squared so you get four 2D coordinates
per marker.

Once you get the markers, you want to get the 3D pose of each marker in the camera space. The
``::registrationCV::SPoseFrom2d`` service takes the previous markers map as input, along with the calibration
of the camera of type ``::arData::Camera``. Each time the map is updated, it fills in a matrix for each identifier,
so here for the tag **101**.

.. code-block:: xml

    <service uid="registrationSrv" type="::registrationCV::SPoseFrom2d" worker="tracking" >
        <in group="markerMap" autoConnect="yes" >
            <key uid="markerMap" />
        </in>
        <in group="camera" >
            <key uid="camera" />
        </in>
        <inout group="matrix" >
            <key uid="markerToCamera" id="104" />
        </inout>
        <patternWidth>60</patternWidth>
    </service>

===============
Augmented view
===============

Now that we get the 3D pose of the marker, it is pretty straightforward to display an object at this location on
top of the video. In the sample, a ``::fwData::Mesh`` is preloaded and contains a cube whose sides have the same
dimensions as the *ArUco* marker.

In the generic scene, a first adaptor is used to display the video on the background layer:

.. code-block:: xml

    <service uid="videoAdp" type="::visuOgreAdaptor::SVideo" autoConnect="yes" >
        <in key="image" uid="sourceFrame" />
        <config layer="video" />
    </service>

Then, to display the cube in 3D, we setup a scene where the cube does not move but the camera receives the inverse
transform of the pose of the marker.

.. code-block:: xml

    <!-- Camera for the 3D layer -->
    <service uid="cameraAdp" type="::visuOgreAdaptor::SCamera" autoConnect="yes" >
        <inout key="transform" uid="cameraToMarker" />
        <in key="calibration" uid="camera" />
        <config layer="default" />
    </service>

    <!-- Cube displayed on top of the marker plane -->
    <service uid="cubeAdp" type="::visuOgreAdaptor::SMesh" autoConnect="yes" >
        <inout key="mesh" uid="cubeMesh" />
        <config layer="default" autoresetcamera="no" />
    </service>

To compute the inverse matrix we use the ``SConcatenateMatrices`` service that can be used to multiply transform
matrices and also to invert them at the same time :

.. code-block:: xml

    <!-- Multiply matrices (here only used to inverse "markerToCamera") -->
    <service uid="matrixReverserSrv" type="::maths::SConcatenateMatrices" >
        <in group="matrix" >
            <key uid="markerToCamera" autoConnect="yes" inverse="true" />
        </in>
        <inout key="output" uid="cameraToMarker" />
    </service>

================
Lens distortion
================

We offer the possiblity to apply the lens distortion correction either to the video or to the 3D rendering. In the
first case we undistort the video, and in the second case we distort the 3D rendering. Undistorting the video is
more common and easier, but in the field of surgery with laparoscopic or endoscopic videos, it may be preferable or
even mandatory to not alter the video image. This is why we give both options.

We can use the same ``::videoCalibration::SDistortion`` service for both cases. Here is the configuration used in the
tutorial to undistort the video:

.. code-block:: xml

    <service uid="undistorterSrv" type="::videoCalibration::SDistortion" >
        <in key="camera" uid="camera" autoConnect="yes" />
        <inout key="map" uid="undistortMap" />
        <mode>undistort</mode>
    </service>

The service takes the calibration camera and it outputs an undistortion map.

A second instance of the service can be used to compute the distortion map:

.. code-block:: xml

    <service uid="distorterSrv" type="::videoCalibration::SDistortion" >
        <in key="camera" uid="camera" autoConnect="yes" />
        <inout key="map" uid="distortMap" />
        <mode>distort</mode>
    </service>

These maps are used to undistort the video or distort the scene directly in the rendering pipeline. To achive this, we
apply a compositor on the video layer and the scene layer, this compositor allows to remap the rendering with
a given map.

.. code-block:: xml

    <service uid="genericSceneSrv" type="::fwRenderOgre::SRender" >
        <!-- It is essential to use the 'sync' mode when doing AR -->
        <!-- In this mode, the renderer will wait for a signal to trigger the rendering -->
        <scene renderMode="sync" >
            <background topColor="#36393E" bottomColor="#36393E" />

            <layer id="video" order="1" compositors="Remap" />
            <adaptor uid="videoAdp" />
            <adaptor uid="undistortAdp" />

            <layer id="default" order="3" compositors="Remap" />
            <adaptor uid="axisAdp" />
            <adaptor uid="cameraAdp" />
            <adaptor uid="cubeAdp" />
            <adaptor uid="distortAdp" />
        </scene>
    </service>

==========
Compositor
==========

To give the map to each compositor, we used a new adaptor ``::visuOgreAdaptor::SCompositorParameter``, it allow to bind a sight data to a compositor.
The first one give the undistortion map to the video layer.

.. code-block:: xml

    <service uid="undistortAdp" type="::visuOgreAdaptor::SCompositorParameter" autoConnect="yes" >
        <inout key="parameter" uid="undistortMap" />
        <config layer="video" compositorName="Remap" parameter="u_map" shaderType="fragment" visible="false" />
    </service>

And the second one give the distortion map to the scene layer.

.. code-block:: xml

    <service uid="distortAdp" type="::visuOgreAdaptor::SCompositorParameter" autoConnect="yes" >
        <inout key="parameter" uid="distortMap" />
        <config layer="default" compositorName="Remap" parameter="u_map" shaderType="fragment" visible="false" />
    </service>

By default, these two compositor are disabled (``visible="false"``), and only one of them can be enabled at the same
time, this is perform by to services ``::gui::action::SSlotCaller``.

.. code-block:: xml

    <service uid="startUndistortAct" type="::gui::action::SSlotCaller" >
        <slots>
            <slot>undistortAdp/show</slot>
            <slot>startDistortAct/setInexecutable</slot>
            <slot>startUndistortAct/hide</slot>
            <slot>stopUndistortAct/show</slot>
        </slots>
    </service>

    <service uid="stopUndistortAct" type="::gui::action::SSlotCaller" >
        <state visible="false" />
        <slots>
            <slot>undistortAdp/hide</slot>
            <slot>startDistortAct/setExecutable</slot>
            <slot>startUndistortAct/show</slot>
            <slot>stopUndistortAct/hide</slot>
        </slots>
    </service>

================
Synchronization
================

The last important part of the tutorial is the synchronization of all these services to get the cube always perfectly
aligned with the *ArUco* marker of the video. What we want to obtain is simple:

- decode a video frame ::videoQt::SFrameGrabber
- detect the tag
- extract the pose
- compute the inverse matrix
- render the scene

However it is not that easy to achieve because the services work independently on different worker threads. So the
execution process rather looks like this :

.. figure:: ../media/TutorialsTuto17SimpleARSync1.png
    :scale: 100
    :align: center

To pipeline all of those services together we use signals and slots.
We first retrieve a frame from the frame timeline filled by the ``SFrameGrabber`` with the help of
the ``::videoTools::SFrameMatrixSynchronizer`` service:

Thanks to the auto connections, the modification of ``sourceFrame`` triggers the video adaptor in
``::visuOgreAdaptor::SVideo`` and in parallel the detection in ``::trackerAruco::SArucoTracker``, which then
modifies the marker map. The modification of the marker map then triggers the computation of the pose in
``::registrationCV::SPoseFrom2d``. Last the modification of the ``markerToCamera`` matrix triggers the computation
of the inverse matrix ``cameraToMarker``.

Now to trigger the rendering of the scene, we simply use the ``SSignalGate`` service which waits on several signals to
be triggered before sending a signal. It is configured by simply giving it the list of signals :

.. code-block:: xml

    <service uid="syncGenericSceneSrv" type="::ctrlCom::SSignalGate" >
        <signal>sourceFrame/bufferModified</signal>
        <signal>cameraToMarker/modified</signal>
    </service>

    <connect>
        <signal>syncGenericSceneSrv/allReceived</signal>
        <slot>genericSceneSrv/requestRender</slot>
        <slot>frameUpdaterSrv/synchronize</slot>
    </connect>

Note that in addition with launching the scene rendering, we also request the frame synchronizer to consume
a new frame, which triggers a new iteration of the pipeline process.

At the end, the execution process looks like this:

.. figure:: ../media/TutorialsTuto17SimpleARSync2.png
    :scale: 100
    :align: center

Please also note that by default, the generic scene renders each time the input of any of there adaptors is modified.
To disable this behavior and synchronize only when requested, we set the ``renderMode`` attribute to ``sync``:

.. code-block:: xml

    <service uid="genericSceneSrv" type="::fwRenderOgre::SRender" >
            <scene renderMode="sync" >
                {...}
            </scene>
        </service>

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tutosimplear

   .. group-tab:: Windows

        .. code::

            bin/tutosimplear.bat

