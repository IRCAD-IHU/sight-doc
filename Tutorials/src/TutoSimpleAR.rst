.. _tutoSimpleAR:

*******************************************************
[*TutoSimpleAR*] Augmented-reality with *ArUco* markers
*******************************************************

This tutorial shows a basic sample of augmented reality.
This exhibits:

- how to detect an *ArUco* marker,
- how to compute the pose of a marker,
- how to make a basic augmented-reality render (we superimpose a plane onto the *ArUco* marker),
- how to undistort a video according to the intrinsic parameters of the camera,
- how to distort a 3D render according to the intrinsic parameters of the camera,
- how to synchronize a video process pipeline with the video playback efficiently, using ``SSignalGate``.

To use this application, you must open a calibration and a video. Samples are provided in the bundle folder
of the appplication, ``share/sight/TutoSimpleAR-0.4`` on Linux/MacOs and ``share\TutoSimpleAR-0.4`` on Windows:

- ``calibration.xml``
- ``aruco_tag.m4v``

.. raw:: html

       <video width="700" controls>
          <source src="https://owncloud.ircad.fr/index.php/s/XabBd5ZiVxfnrVh/download" >
          Your browser does not support the video tag.
       </video>

Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`tuto08`

Tracking
=================

Tag detection is done with the ``::trackerAruco::SArucoTracker`` service, which takes a video frame
as input, and fills in a map of identified *ArUco* markers. You only have to specify which marker identifier you
want to retrieve, here we choose the tag **101** because it is the one seen in the video sample.

.. code-block:: xml

        <service uid="tracker" type="::trackerAruco::SArucoTracker" worker="tracking">
            <in key="camera" uid="camera" />
            <inout key="frame" uid="sourceFrame" autoConnect="yes" />
            <inout group="markerMap">
                <key uid="markerMap" /> <!-- timeline of detected tag(s) -->
            </inout>
            <track>
                <!-- list of tag's id -->
                <marker id="101" />
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

        <!-- Computes the pose of the camera with tag(s) detected by aruco -->
        <service uid="registration" type="::registrationCV::SPoseFrom2d" worker="registration">
            <in group="markerMap" autoConnect="yes">
                <key uid="markerMap" />
            </in>
            <in group="camera">
                <key uid="camera" />
            </in>
            <inout group="matrix">
                <key uid="markerToCamera" id="101" />
            </inout>
            <patternWidth>60</patternWidth>
        </service>

Augmented view
===============

Now that we get the 3D pose of the marker, it is pretty straightforward to display an object at this location on
top of the video. In the sample, a ``::fwData::Mesh`` is preloaded and contains a cube whose sides have the same
dimensions as the *ArUco* marker.

In the generic scene, a first adaptor is used to display the video on the background layer:

.. code-block:: xml

        <service uid="videoAdpt" type="::visuVTKARAdaptor::SVideo" autoConnect="yes">
            <in key="frame" uid="finalFrame" />
            <config renderer="video" />
        </service>

Then, to display the cube in 3D, we setup a scene where the cube does not move but the camera receives the inverse
transform of the pose of the marker.

.. code-block:: xml

        <!-- Camera for the 3D layer -->
        <service uid="cameraAdpt" type="::visuVTKARAdaptor::SCamera" autoConnect="yes">
            <in key="transform" uid="cameraToMarker" />
            <in key="camera" uid="camera" />
            <config renderer="default" />
        </service>

        <!-- Cube displayed on top of the marker plane -->
        <service uid="cubeAdpt" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
            <in key="mesh" uid="cubeMesh" />
            <config renderer="default" autoresetcamera="no" color="#ffffffda"/>
        </service>

To compute the inverse matrix we use the ``SConcatenateMatrices`` service that can be used to multiply transform
matrices and also to invert them at the same time :

.. code-block:: xml

        <!-- Multiply matrices (here only used to inverse "markerToCamera") -->
        <service uid="matrixReverser" type="::maths::SConcatenateMatrices">
            <in group="matrix">
                <key uid="markerToCamera" autoConnect="yes" inverse="true" />
            </in>
            <inout key="output" uid="cameraToMarker" />
        </service>

Lens distortion
================

We offer the possiblity to apply the lens distortion correction either to the video or to the 3D rendering. In the
first case we undistort the video, and in the second case we distort the 3D rendering. Undistorting the video is
more common and easier, but in the field of surgery with laparoscopic or endoscopic videos, it may be preferable or
even mandatory to not alter the video image. This is why we give both options.

We can use the same ``::videoCalibration::SDistortion`` service for both cases. Here is the configuration used in the
tutorial to undistort the video:

.. code-block:: xml

        <!-- Undistort the video frame -->
        <service uid="undistorter" type="::videoCalibration::SDistortion" worker="distortion">
            <in key="camera" uid="camera" />
            <in key="input" uid="sourceFrame" />
            <inout key="output" uid="finalFrame" />
            <mode>undistort</mode>
        </service>

The service is put in a dedicated worker thread to avoid overloading the main thread of the application. It takes
the calibration camera and the distorted original image as input. It outputs a corrected image. It is toggled thanks
to a slot called ``changeState``. If it is not enabled, it simply copies the original image onto the output image.

A second instance of the service can be used to distort the video:

.. code-block:: xml

        <!-- Distort the offscreen render of the 3D scene -->
        <service uid="distorter" type="::videoCalibration::SDistortion" worker="distortion">
            <in key="camera" uid="camera" />
            <in key="input" uid="offscreen3DImage" autoConnect="yes" />
            <inout key="output" uid="distorted3DImage" />
            <mode>distort</mode>
        </service>

However to achieve this, we need first to get the 3D rendering. We do this in a separate generic scene configured
with an offscreen buffer as *inout* data :

.. code-block:: xml

        <service uid="offscreenRender" type="::fwRenderVTK::SRender">
            <inout key="offScreen" uid="offscreen3DImage" />
            ...
        </service>

Synchronization
================

The last important part of the tutorial is the synchronization of all these services to get the cube always perfectly
aligned with the *ArUco* marker of the video. What we want to obtain is simple:

- decode a video frame ::videoQt::SFrameGrabber
- detect the tag
- extract the pose
- compute the inverse matrix
- compute the distortion
- render the scene

However it is not that easy to achieve because the services work independently on different worker threads. So the
execution process rather looks like this :

.. figure:: ../media/sync1.png
    :scale: 100
    :align: center

To pipeline all of those services together, we use signals and slots. We first retrieve a frame from the frame timeline
filled by the ``SFrameGrabber`` with the help of the ``::videoTools::SFrameMatrixSynchronizer`` service:

.. code-block:: xml

        <service uid="synchronizer" type="::videoTools::SFrameMatrixSynchronizer" >
            <in group="frameTL">
                <key uid="frameTL" autoConnect="yes"/>
            </in>
            <inout group="image">
                <key uid="sourceFrame" />
            </inout>
            <tolerance>100</tolerance>
            <framerate>0</framerate>
        </service>

Thanks to the auto connections, the modification of ``sourceFrame`` triggers the distortion in
``::videoCalibration::SDistortion`` and in parallel the detection in ``::trackerAruco::SArucoTracker``, which then
modifies the marker map. The modification of the marker map then triggers the computation of the pose in
``::registrationCV::SPoseFrom2d``. Last the modification of the ``markerToCamera`` matrix triggers the computation
of the inverse matrix ``cameraToMarker``.

Now to trigger the rendering of the scene, we simply use the ``SSignalGate`` service which waits on several signals to
be triggered before sending a signal. It is configured by simply giving it the list of signals :

.. code-block:: xml

        <!-- Wait for the undistortion and the matrix inversion to be finished -->
        <service uid="syncGenericScene" type="::ctrlCom::SSignalGate">
            <signal>finalFrame/bufferModified</signal>
            <signal>cameraToMarker/modified</signal>
        </service>

        ...

        <!-- When the undistortion and the matrix inversion are done, trigger the rendering -->
        <!-- then process a new frame -->
        <connect>
            <signal>syncGenericScene/allReceived</signal>
            <slot>genericScene/requestRender</slot>
            <slot>synchronizer/synchronize</slot>
        </connect>

Note that in addition with launching the scene rendering, we also request the frame synchronizer to consume
a new frame, which triggers a new iteration of the pipeline process.

At the end, the execution process looks like this:

.. figure:: ../media/sync2.png
    :scale: 100
    :align: center

Please also note that by default, the generic scene renders each time the input of any of its adaptors is modified.
To disable this behavior and synchronize only when requested, we set the ``renderMode`` attribute to ``sync``:

.. code-block:: xml

        <service uid="offscreenRender" type="::fwRenderVTK::SRender">
           <scene renderMode="sync">
            ...
           </scene>
        </service>


Run
===

To run the application, you can run the following line in the install or build directory:

.. code::

    bin/tutosimplear # Linux/MacOs
    bin/tutosimplear.bat # Windows


